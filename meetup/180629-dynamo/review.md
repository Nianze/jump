# Dynamo

As a highly available and scalable distributed data store, the design principles of Dynamo features high availability for writes (via reconciliation during reads), incremental scalability (consistent hashing), symmetry, decentralization, and heterogeneity (via virtual nodes).

#### The summary of techniques used in _Dynamo_ and their advantages

**Problem** | **Technique** | **Advantage**
---         |   ---         |      ---
Partitioning|Consistent Hashing|Incremental Scalability
High Availability|Vector clocks with reconciliation during reads|Version size is decoupled from update rates.
Handling temporary failures|Sloppy Quorum and hinted handoff|Provides high availability and durability guarantee when some of the replicas are not available.
Recovering from permanent failures|Anti-entropy using Merkle trees|Synchronizes divergent replicas in the background.
Membership and failure detection|Gossip-based membership protocol and (local notion) failure detection|Perserves symmetry and avoids having a centralized registry for storing membership and node liveness information.

