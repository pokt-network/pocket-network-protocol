%% Not including all actors (fisherman, applications, servicers, fisherman)

```mermaid
flowchart LR
    subgraph Node
        C[Client]
        KVS[Key Value Store]
        SQL[(SQL DB Engine)]
        T{Merkle Patricia Trie}
        BH(Block Header)
        B[Block w/ ]

        C -- History --> SQL
        T -- Security & Integrity --> BH
    end
```

```mermaid
flowchart LR
    subgraph Data Storage
        C
        KVS[(Key Value Store)]
        T[(Modified Merkle\nPatricia Trie)]
        SQL[(SQL DB Engine)]
        BH(Block Header)

        T --> BH
    end
```

```mermaid
flowchart LR
    subgraph Block Header
        PrevHash
        Transactions
        QuorumCertificate
        OBH[...]
        subgraph State Trie Roots
            VR[Validators Root]
            AR[Applications Root]
            OSTR[...]
        end
    end

    %% Source of Truth
    subgraph "Protobuf Schema"
        VP([Validator])
        AP([Application])
        OPS([...])
    end

    %% Client Customizable
    subgraph "SQL Schema"
        VT[(Validators Table)]
        AT[(Applications Table)]
        OSS[(...)]
    end

    VP -- Serialization & \n Trie Insertion --> VR
    AP -- Serialization & \n Trie Insertion --> AR
    OPS -- Serialization & \n Trie Insertion --> OSTR

    VP -- Schema Definition & \n DB Insertion --> VT
    AP -- Schema Definition & \n DB Insertion --> AT
    OPS -- Schema Definition & \n DB Insertion --> OSS
```

```mermaid
pie title Pets adopted by volunteers
    "Dogs" : 386
    "Cats" : 85
    "Rats" : 15
```

```mermaid
flowchart TB
    subgraph Fisherman Pool
        F1(Fisherman 1)
        F2(Fisherman 2)
        F3(Fisherman 3)
        F1 <-- 👀 --> F2
        F2 <--> F3
        F1 <--> F3
    end
```

```
flowchart LR
    subgraph FP["Fisherman Pool (Chain, GeoZone)"]
        F1("Fisherman 1\n📝💯")
        F2("Fisherman 2\n📝💯")
        F3("Fisherman 3\n📝💯")
        F1 <-- "👀" --> F2
        F2 <-- "👀" --> F3
        F1 <-- "👀" --> F3
    end

    subgraph SP["Servicer Pool (Chain, GeoZone)"]
        SN1("Service Node 1\n📝💯")
        SN2("Service Node 2\n📝💯")
        SN3("Service Node 3\n📝💯")
    end

    subgraph VP["Validator Pool"]
        V1("Validator 1")
        V2("Validator 2")
        V3("Validator 3")
        V1 <-- "👀" --> V2
        V2 <-- "👀" --> V3
        V1 <-- "👀" --> V3
    end

    subgraph OP["Other Pools"]
        DAO["👀 DAO Pool 👀"]
        FEE["Fee Pool"]
        APP["App Pool\n(Chain, Geozone)"]
    end

    VP --> OP
    VP --> FP
    VP --> SP
    FP --> SP

```
