### Cassandra follow-ups

**Q: Cassandra's super columns, super column families, differentiate with the concept in BigTable?**

A: First off, [super column / super column family are long gone]((https://docs.datastax.com/en/articles/cassandra/cassandrathenandnow.html)). Column families are replaced with "tables".
Back in the days when they had such concepts, it seems that super columns are a (schema'd) collection of columns, which seems to be the same as a column whose content type is a map with predetermined keys.
Back in the days Cassandra did not have Collections data type available for column types, (neither do they have composite keys it seems), now they have both and there does not seem to be a need for super columns.
A super column family seems to be a column family whose columns can be super columns.
The concepts themselves don't seem to be particularly constructive, if we were to compare with BigTable, a super column seems to correspond to a column family in BigTable, with the difference being internals of super columns follow a schema while inside BigTable's column family there is no schema.

**Q: What is [gossip protocol](https://docs.datastax.com/en/cassandra/2.1/cassandra/architecture/architectureGossipAbout_c.html), membership, failure detection and snitch**

A: Gossip is a periodic, pair-wise, unreliable, state information exchanges between nodes participating in a distributed system. Messages can expect some redundancy, and the choice of peers to talk to can be random.
Cassandra uses gossip to sync up on nodes' states. In lieu of a traditional failure detection by failing consecutive heartbeats, Cassandra uses an [accrual detection mechanism](https://docs.datastax.com/en/cassandra/2.1/cassandra/architecture/architectureDataDistributeFailDetect_c.html) to calculate a per-node threshold that takes into consideration network performance, workload, and historic conditions.
Cassandra gossip messages contain versioned Cassandra application components states, end point states, and end point states map (transitive information this node heard about is passed on).
Every second a node picks a random live node to gossip to, and a random unreachable node to gossip to with probability _P_. If the live node is not a seed or the number of live nodes is less than number of seeds, gossip to a seed with probability _P1_.
[More details](https://wiki.apache.org/cassandra/ArchitectureGossip).

A [snitch](https://docs.datastax.com/en/cassandra/2.1/cassandra/architecture/architectureSnitchesAbout_c.html) decides which datacenters and racks nodes belong to, and the performance of certain replicas. Cassandra needs topology information in order to satisfy user's replica configuration (say, not placing two copies in the same rack), and performance is monitored to decide the best replica to satisfy this read request (one replica provides full data, while others provide checksums of the same data).
It seems data gathered by snitches are exchanged via gossip.

**Q: partition key and clustering key?**

A: [partition key](https://docs.datastax.com/en/cql/3.1/cql/ddl/ddl_compound_keys_c.html): decides which node stores the data. Clustering key: decides how data is sorted within a partition.
(murmur3(partition key) ==> hash result to decide where in the ring the record should be stored)
By default, a PRIMARY KEY (key1, key2, key3) statement creates key1 as partition key, and the composite (key2, key3) as clustering key. One can specify a composite partition key as well.
Such queries are efficient:
- equality on partition key
- equality on partition key + range query on clustering key

**Q: what is [partition summary](https://docs.datastax.com/en/cassandra/3.0/cassandra/dml/dmlAboutReads.html) and how does it help?**

A: (As Pranav suggested) partition summary is an in-memory sampled partition index (sorted by hash(partition key) and lives on disk) per SSTable.
Each summary samples one partition index, mapping to the file offset of the sampled keys, such that when trying to read from a particular key, only a scan on a small range in the file is needed (as identified by the partition summary).

```
data models: column family, super column, super column family, difference and why?

hinted handoff in write

node departure in Chord

The key in memtable is a primary key or the partition key?
You do need to query SStable in regular reads
```
