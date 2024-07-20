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

- crash -- or crash loop
- server down
- data corruption
- query of death
- broken dependency
- cascading failure
- DOS attack
- heisenbug: when the system fails in prod but we can't make it to fail in testing
- customer reported failure
- data loss
- time travel: system depends on the clock in the time of the day, but for some reason it seems it has gone backwards!
- owned: we had security breach, the system or some nodes of the system appear to be owned!
- natural disaster
- cause infra failure: the system itself caused an infra failure
- operator error
- runaway automation: automation breaks stuff
- certificate expires
- fail to pay bills
- non-linear response to change
- performance cliffs
- very important user left the company
- non-hermetic builds
- firewalls
- kernel memory leak: your processes are running on kernels which are leaking memory and they randomly crash
- isolation failure: for example, your processes are running on a machine shared with other people's processes and their
  perf is causing random perturbation in your processes perf
- hash collision: you're running your computation over very large amount of data. You discover that some of your data is
  corrupted but the checksums still check out and say the data isn't corrupted. You found a hash collision.
- incorrect algorithm: we discovered that the algo isn't correct for all inputs. So the algo is wrong.

## L5: The many types of fail
