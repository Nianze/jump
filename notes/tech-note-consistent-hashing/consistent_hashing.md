# Consistent Hashing

### Property

Given K number of keys and n number of slots, a change in the number of slots (hashtable resize) will cause on average K/n keys to be relocated, instead of in traditional hashing where almost all keys needs to be relocated.

### Usage

To reduce impact of partial system failures in large applications.

DHT uses consistent hashing to partition a keyspace.

### Technique

Map hashed values on a hash ring, instead of a linear space: each hash(key) corresponds with some values (angles) in the hash ring. Each server also has its hash(ID) mapped to some angles in the hash ring, and algorithm dictates that every key should fall into the first server in the ring that comes after it in a clockwise / counter-clockwise fashion.

In code, when looking for which server holds a particular key, could do a binary search on servers ordered by hash(ID).

To distribute keys evenly, each host corresponds with multiple tags (say, ID + number), and a hash(key) is hosted by the server that has the tag whose hash(tag) sits cloest to hash(key) in a clockwise / counter-clockwise fashion. How many tags a server corresponds to can be arbitrary, thus accounting for difference in server processing power.

With this, if a server fails, only keys that need to be relocated are those that were previously hosted by this server.

### Trivia

Authors of the 1997 consistent hashing paper went on to found Akamai and build big CDNs with consistent hashing.

### Further reading

https://www.akamai.com/es/es/multimedia/documents/technical-publication/consistent-hashing-and-random-trees-distributed-caching-protocols-for-relieving-hot-spots-on-the-world-wide-web-technical-publication.pdf
