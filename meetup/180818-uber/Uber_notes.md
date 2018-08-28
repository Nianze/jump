Uber's architecture:

Ringpop:
Implements a variation of **SWIM** (Scalable Weakly-consistent Infection-style Process Group Membership) protocol.

    1. Information dissemination:
    A cluster with two nodes: A and B. A is pinging B and B is pinging A. 
    Then a third node, C, joins the cluster after pinging B. 
    At this point B knows about C, but A does not. The next time B pings A, 
    it will disseminate the knowledge that C is now part of the cluster.

    2. Failure detection:
    Each node randomly selects 1 peer and sends a PING message to it. 
    Once a node receives a PING message it responds to the sender with an ACK message.
    If a ACK is not received within some predefined time span, the node switches to indirect ping; 
    It randomly selects k other peers and asks each one of them to ping that node and forward the ACK back. 
    Only if a node has not received neither the original ACK not any of the k indirect ACKs it assumes a peer is dead.

    3. Sync:
    A membership list contains the addresses and statuses (alive, suspect, faulty, etc.) of the instances. 
    It also contains additional metadata like the incarnation number, which is the logical clock. 
    All the information is combined and we compute a checksum from it.

    4. Consistent hashing: [farmhash](https://github.com/google/farmhash)
    Data structure for ring: Red-Black Tree, the ring is for maintaining the hash for WORKERS (DHT at application level)
    Reliable background operations: maintain a (vnode, key) mapping. One way to do this is key modulo the number of vnode.
    Example:
    Worker A received request and started a timer, it wrote key to vnode set in database.
    Then A crashed. Worker B detected a membership change in the ring, it scanned the entire vnode space,
    and when it found itself, it would restore the timer status of keys corresponding to its vnode.

    5. Actor model:
    Every actor in the system has a home (a node in the cluster). 
    For every actor, there is a mailbox. Requests get pulled off the mailbox one by one. 
    Processing a request may result in new requests being sent or new actors being created. 
    Each request that is processed one by one may result in some other request to another service, 
    or a request for more actors to be spun up.

    6. Flap damping:
    Identify and evict bad nodes from a cluster. Example: A pings B, and B responds. 
    Then, in the next round of the protocol, A pings B again but this time B is down. 
    Then in the next round, A pings B, but this time B is up again.
    Solution: When we detect a flap, we penalize the bad actor. Every node stores a penalty for every other node in the cluster.

    7. Forwarding details: TChannel protocol [TChannel](https://github.com/uber/tchannel/blob/master/docs/protocol.md)


Uber's design in brief:

    1. Requirements:
    Drivers need to notify the service their locations regularly, let's assume 4s.
    Passengers need to be able to see all the available drivers nearby.
    Customer requests drivers, service does matching. (Customers/Drivers can cancel the orders.)
    Once the matching is done, customers and drivers are able to see regular update of each other's locations.
    Driver start/end a trip. Customers and drivers can rate each other when the trip finishes.

    2. Capacity:
    170k peak QPS: [qps](https://eng.uber.com/go-geofence/)

    3. Difficulties:
    Find nearby drivers: we can use geohash (https://en.wikipedia.org/wiki/Geohash)
    Update locations of drivers: Probably Redis
    Location: Key: geohash Value: driver1, driver2, ...
    Driver: Key: driver ID Value: lat, lng, updated time, status

    4. Database sharding: City
    Two level approach: First scan all the cities then the geofences inside one city
    Find out whether a point is inside a polygon: Ray Casting


References for further study:

    1. [Ringpop on Uber's engineering blog](https://eng.uber.com/intro-to-ringpop/)
    2. [Ringpop on Youtube](https://www.youtube.com/watch?v=OQyqJWQHp3g)
    3. [Geofence](https://eng.uber.com/go-geofence/)
    4. [Scaling Uber's Real-time Market Platform](https://www.infoq.com/presentations/uber-market-platform)
    5. [UberRUSH](https://qconnewyork.com/ny2015/system/files/presentation-slides/uberRUSH.pdf)
