# Replace Badger KVStore <!-- omit from toc -->

*tl;dr - Replace the key-value store backing the Sparse Merkle Tree (SMT) from BadgerDB to Postgres.*

* Status: _Withdrawn_
* Deciders: Protocol Network Team
* Date: 2023-04-25

## Table of Contents <!-- omit from toc -->
<!-- TOC -->

- [Technical Story](#technical-story)
- [Summary](#summary)
- [Context and Problem Statement](#context-and-problem-statement)
- [Design Goals](#design-goals)
- [Decision Drivers](#decision-drivers)
- [Considered Options](#considered-options)
- [Decision Outcome](#decision-outcome)
  - [Positive Consequences](#positive-consequences)
  - [Negative Consequences](#negative-consequences)
- [Pros and Cons of the Options](#pros-and-cons-of-the-options)
  - [\[Option 1\] - Use Postgres](#option-1---use-postgres)
  - [\[Option 1.1\] - Store Keys In HSTORE](#option-11---store-keys-in-hstore)
  - [\[Option 1.2\] - Store Keys In JSONB](#option-12---store-keys-in-jsonb)
  - [\[Option 1.3\] - Store Keys In KV Table](#option-13---store-keys-in-kv-table)
  - [\[Option 2.1\] - Make KV Store Handle Atomic Guarantees](#option-21---make-kv-store-handle-atomic-guarantees)
  - [\[Option 3\] - Manually Cleanup Leftover Keys](#option-3---manually-cleanup-leftover-keys)
  - [\[Option 4\] - Do Nothing](#option-4---do-nothing)
- [Decision](#decision)
- [Next Steps](#next-steps)
- [Links](#links)
<!-- TOC -->

## Technical Story

* [Pocket Network 1.0 Persistence Module Specification](https://github.com/pokt-network/pocket-network-protocol/blob/main/persistence/README.md#1-why-not-just-store-the-state-in-a-tree-like-every-other-blockchain)
* [Issue #327 - Implement KISS Savepoints & Rollbacks](https://github.com/pokt-network/pocket/issues/327)
* [PR #657 - KV POC](https://github.com/pokt-network/pocket/pull/657)
* [PR #645 - JSON POC](https://github.com/pokt-network/pocket/pull/645)
* [#692 - Explore Replacing the SMT Key-Value Store From Badger To Postgres](https://github.com/pokt-network/pocket/issues/692)

## Summary

This ADR explores replacing the Badger KV store with Postgres for the Sparse Merkle Tree (SMT) representation of transactions in the Pocket Network protocol. The main issue is that Badger stores key/value pairs, which are not synchronized with Postgres during atomic rollbacks, causing potential inconsistencies between Badger and Postgres, including orphaned keys in the Badger KV store.

The decision drivers are simplifying transaction handling for savepoints and rollbacks and elimination of a dependency on Badger, which decreases maintenance and complexity costs. The considered options include using PostgreSQL's HSTORE, JSONB, or a plain key-value table, with the outcome still to be determined. The documentation outlines the pros and cons of each option, including atomic transactions, index capabilities, performance, and Postgres-specific behavior. The do-nothing option is also discussed but ultimately is not the preferred choice as it does not allow for synchronization of atomic backups or sync.

## Context and Problem Statement

When applying blocks, Postgres validates the block contents. However, Badger stores key/value pairs that make up the SMT representation of transactions for each block during block application. When a block fails validation and thus application, i.e. Postgres declares the data invalid, a rollback is initiated in Postgres, but the key/value pairs of that failed transaction are still left in Badger. The problem lies in the KVStore interface. The KVStore interface does not currently expose commit handling but acts as a subordinate component of an atomic persistent context. This means that its wrapper, the persistence context, can't itself provide specific atomic guarantees.

## Design Goals

The goal is synchronizing the atomic application or reversal of block data between both Postgres and Badger such that we are not left with orphaned keys or rows in the KVStore or the Postgres database.

## Decision Drivers

* The ability to handle atomic rollbacks is crucially important, but the split between Badger and Postgres means one more variable to handle in an already complex process of advancing and rolling back state when necessary.
* Simplifying this feature could eliminate a dependency and decreases maintenance costs.

## Considered Options

1. Use Postgres for atomic guarantees
  1.1. Store in HSTORE
  1.2. Store in JSONB
  1.3. Store in Plain Table
2. Handle atomic guarantees at the KVStore
3. Manually cleaning up orphaned keys

## Decision Outcome

This ADR is withdrawn. The decision has been made to focus on refactoring the KVStore interface to guarantee atomic block application. That relates to exploration in Option 2 in this document.

### Positive Consequences

* Allows us to remove a dependency on Badger.
* Simplifies transaction handling for savepoints and rollbacks.

### Negative Consequences

* Badger is highly performant and the KV store in place currently is very efficient. Changing this part of the system could mean changing back in the future in the face of performance issues.
* Any of the three Postgres KV store implementations outlined below requires a migration to complete.

```go
// TODO(#77): Enable proper up and down migrations
func initializeDatabase(conn *pgxpool.Conn) error {
  // TODO: IN THIS COMMIT initialize key value store if it doesn't exist
  // Initialize the tables if they don't already exist
  if err := initializeAllTables(context.TODO(), conn); err != nil {
    return fmt.Errorf("unable to initialize tables: %v", err)
  }
  return nil
}
```

## Pros and Cons of the Options

### [Option 1] - Use Postgres

This option broadly means we stop storing key value pairs in Badger and instead store them all in Postgres. Postgres supports several technologies that would facilitate this approach, each of which we examine in-depth below.

Using Postgres in general is...

* Good, because it's very well supported and documented.
* Good, because it's generally good enough.
* Bad, becaues it's a heavy dependency.

### [Option 1.1] - Store Keys In HSTORE

Option 1 would be to use the PostgreSQL extension [HSTORE](https://www.postgresql.org/docs/current/hstore.html#id-1.11.7.27.7).
[HSTORE](https://www.postgresql.org/docs/current/hstore.html#id-1.11.7.27.7) supports multiple types of indexes, including btree, GIN, and GIST indexes.
This option was considered in the [Persistence Module Specification](https://github.com/pokt-network/pocket-network-protocol/tree/main/persistence#2-what-kind-of-key-value-store-database-engine-is-going-to-back-the-merkle-tree).
HSTORE has the same transaction gurantees that table modification does in Postgres.

```sql
BEGIN; -- start a transaction
UPDATE my_table SET my_hstore_column = my_hstore_column || 'new_key=>'new_value';
-- perform some updates to the hstore column
COMMIT; -- commit the transaction, making all the changes permanent

-- or, if you want to rollback the changes:
ROLLBACK; -- rollback the transaction, undoing all the changes made within it
```

It also supports prefix queries.

```sql
SELECT key, value
FROM my_table, each(hstore_column) AS h(key, value)
WHERE key LIKE 'prefix%';
```

* Good, because it's meant as a KV store.
* Good, because it supports prefix queries.
* Good, because it can be indexed like any column.
* Good, because it guarantees atomic transactions.
* Bad, because it's Postgres specific which could impose limitations on us in the future or be a source of *unknown unknowns*.
* Bad, because it's an extension, not a native feature, which adds complexity and maintenance of its own.
* Bad, because it uses a novel syntax.
* Bad, because it might not have the same performance capabilities as Badger.
* Bad, because it requires a migration.

### [Option 1.2] - Store Keys In JSONB

Option 2 would create a JSONB column with a key of TEXT and a value of JSONB. This allows more complex structures in the values column.

```sql
SELECT key, value
FROM my_table, jsonb_each(jsonb_column) AS j(key text, value jsonb)
WHERE key LIKE 'prefix%';
```

* Good, because it can be indexed like a normal table.
* Good, because it can handle prefix queries.
* Good, because it guarantees atomic transactions.
* Good, because it easily handles more complex values.
* Bad, because we want strict hash comparisons to continue working and JSONB interpretation could have unforeseen consequences.
* Bad, because it might not have the same performance capabilities as Badger.
* Bad, because it's Postgres specific which could impose limitations on us in the future or be a source of *unknown unknowns*.
* Bad, because it requires a migration.

### [Option 1.3] - Store Keys In KV Table

Option 3 would dump everything into typical TEXT types in a standard table named `transaction`.

* Good, because it's simple and straight-forward table access. JSONB and HSTORE have new, Postgres specific syntax and behavior.
* Good, because it stays within standard SQL features, and doesn't utilize Postgres specific behavior.
* Good, because it handles prefix queries.
* Bad, because it lacks similar performance capabilities compared to Badger.
* Bad, because it requires a migration.

### [Option 2.1] - Make KV Store Handle Atomic Guarantees

Option 2 would involve extending the `KVStore` interface so that it can be atomically controlled by the caller (i.e. TxIndexer, block store, or the state trees) either by having it directly extend the Rollback & Savepoint interface, as demonstrated below, or by having it manage its own rollbacks.

```go
func (indexer *txIndexer) Rollback([]byte) error {
  return fmt.Errorf("not impl")
}
func (indexer *txIndexer) Savepoint() ([]byte, error) {
  return nil, fmt.Errorf("not impl")
}
```

This option is similar to [PR #657 - KV POC](https://github.com/pokt-network/pocket/pull/657) but different in a key way: it addresses the core design misalignment between the `PersistenceRWContext` and the `KVStore` causing the dissonance in the first place.

* Good, because it avoids migrations.
* Good, because it clears up KVStore's responsiblity boundary as a component in this system.
* Good, because it takes advantage of existing Badger features.
* Bad, because it could be more complex than just passing it off to Postgres.

### [Option 3] - Manually Cleanup Leftover Keys

This option is to manually remove or delete keys from Badger that are added in the middle of a rollback.

* Bad, because manually handling cases like this is error prone and this issue directly effects chain integrity.
* Good, because it requires smaller and fewer changes to implement.

### [Option 4] - Do Nothing

One should always consider the option of doing nothing. In this case, doing nothing results in orphaned keys being left in the Badger KV store.

* Good, because it is always easy to continue in the same pattern.
* Good, because it fully leverages the performance of Badger.
* Good, because it avoids a new table in the Postgres database, which avoids a migration.
* Bad, because it doesn't allow us to easily synchronize an atomic backup or sync and leads to continued orphaned keys in the database.
* Bad, because it currently leaves orphaned keys in the database.
* Bad, because it could cause issues in block validation and normal downstream node operations after an error occurred.

## Decision

The decision to replace BadgerDB with PostgreSQL was ultimately abandoned because it would not solve the core problem of atomic handling for each constituent part. Instead, a refactor is the only way to truly handle the atomic behavior necessary for confidently applying a unit of work to the persistence layer. If in the future this simplification is again desired, it would still need to be refactored such that the component allowed the caller to handle rollback scenarios.

## Next Steps

* Begin refactoring the KVStore to handle atomic all-or-nothing commits.
* Refactor the sub-stores attached to the PersistenceContext to expose fine-grained commit controls.A

## Links

* [Pocket Network 1.0 Persistence Module Specification](https://github.com/pokt-network/pocket-network-protocol/blob/main/persistence/README.md#1-why-not-just-store-the-state-in-a-tree-like-every-other-blockchain)
* [Issue #327 - Implement KISS Savepoints & Rollbacks](https://github.com/pokt-network/pocket/issues/327)
* [PR #657 - KV POC](https://github.com/pokt-network/pocket/pull/657)
* [PR #645 - JSON POC](https://github.com/pokt-network/pocket/pull/645)
* [#692 - Explore Replacing the SMT Key-Value Store From Badger To Postgres](https://github.com/pokt-network/pocket/issues/692)
* [HSTORE](https://www.postgresql.org/docs/current/hstore.html#id-1.11.7.27.7)
* [Persistence Module Specification](https://github.com/pokt-network/pocket-network-protocol/tree/main/persistence#2-what-kind-of-key-value-store-database-engine-is-going-to-back-the-merkle-tree)
* [Badger - Managing Transactions Manually](https://dgraph.io/docs/badger/get-started/#managing-transactions-manually/)
