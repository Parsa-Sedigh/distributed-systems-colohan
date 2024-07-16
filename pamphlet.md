## L3: How to learn distributed systems

### how to use time in a distributed world - Time

My watch isn't synchronized with your watch. So for example this moment, we may disagree about what time it is.
This matters whenever you have events happening on multiple machines and you care about the ordering between those
events.

For example the comments appear in one order for a user in china and another order for a user in north america. So they
need to agree about the time.

This is hard to get it right. So ignore wall clocks and use `virtual clocks` which is the relationship between
various events on various machines.

### how to get agreement? - Consensus

Consensus is not only about agreeing on what happened. The machines should also agree on what happened even after the
whole system exploded and we bring it all up again. So consensus algos attempt to not only agree upon the ordering
between events but also agree upon it in a **durable** way such that the ordering is written out to durable storage.

Two popular protocols for this(for consensus which means agreeing upon the ordering and making that ordering
durable):

- paxos(hard to understand)
- raft

### how to persist data? - distributed storage

Usually on disk or flash.

### how to secure your system - security

### how to operate your distributed system - the art of SRE

## L4: What could go wrong?