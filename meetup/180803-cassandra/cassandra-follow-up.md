### Cassandra follow-ups

Q: Cassandra's super columns, super column families, differentiate with the concept in BigTable?
A: First off, super column / super column family are long gone. Column families are replaced with "tables". \[1\]
Back in the days when they had such concepts, it seems that super columns are a (schema'd) collection of columns, which seems to be the same as a column whose content type is a map with predetermined keys.
Back in the days Cassandra did not have Collections data type available for column types, (neither do they have composite keys it seems), now they have both and there does not seem to be a need for super columns.
A super column family seems to be a column family whose columns can be super columns.
The concepts themselves don't seem to be particularly constructive, 

1. [Facebook’s Cassandra paper, annotated and compared to Apache Cassandra 2.0](https://docs.datastax.com/en/articles/cassandra/cassandrathenandnow.html)

Q: [gossip protocol](https://docs.datastax.com/en/cassandra/2.1/cassandra/architecture/architectureGossipAbout_c.html), membership, failure detection and snitch

Q: [partition summary](https://docs.datastax.com/en/cassandra/3.0/cassandra/dml/dmlAboutReads.html)
