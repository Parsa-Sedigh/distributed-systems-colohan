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

### Network failures vs node failures

TCP/IP: it gives us this: your data goes from one computer to another, or it doesn't. It deals with multipath effects,
congestion and having to retransmit the lost packets and it does a little bit of protection against data corruption.
But it doesn't handle security.

But SSH does provide security: it provides an encrypted tunnel with authentication between two nodes.

### Network partition

Partition: at least 2 components of system continue to run, but can't communicate to each other.

### Why do we care about (network) partitions?

We tend not to care about partitions if our distributed system is executing a read-only workload. If we're only serving
reqs and not changing any state in our distributed system, there's very little node for our nodes to communicate with
each other.
So if they can't communicate with each other, who cares! But most distributed systems aren't like that. They have state
and that state can change.

When a partition occurs, there are two approaches to deal with it:

- tolerating it
- not tolerating it(maintaining a strictly consistent state)

Shared state diverges. So if the distributed system is partitioned, then the state can diverge. For example, users can't
see each others comments on the same post. So thread of conversation could diverge.

We can actually deal with this. If you write some code that when the partition heals itself and we get one system again,
you identify the conflicts and change the state to make it right. This is called tolerating a inconsistent state and can
be hard or easy depending on the app.

Another approach is maintaining a strictly consistent state. To do this, we need to figure out when your system has
partitioned and make sure all of the sub-graphs except one, disables writes to any item when the system is partitioned,
so that you don't end up with any item in inconsistent state.

So now if one nodes goes down that the other nodes depend on, all the dependent nodes must shutdown as well to maintain
consistency. So the outage of one node can take down multiple nodes.

So to maintain consistency, we have to have a lower availability(in comparison to allowing relaxed consistency model).

This trade-off is the CAP principle.

### Detecting partitions

Q: How could we design an algo that gives us the ability to detect partitions and do the right thing?

A: Every node needs to periodically send some msg to every other node, whether it's a heartbeat, a ping msg or ... . It
can count the responses if it's less than or equal to the total nodes of the system, the current node switches itself to
read-only mode until the node or network heals.

Note: The failures are because of node failure or network link failure.

1. ping all N nodes
2. count responses => R
3. if R <= floor(N/2) => curr node goes to read-only mode

Q: Why the floor(N/2) magic number?

A: If we used any smaller number, then as an individual node talking to other nodes in ds system, I can't tell apart a
scenario from when I'm in a partition that's less than half and the rest of the system is down VS I'm in a partition
that is less than half the size of the system and the rest of system is up and has a bigger group of nodes operating in
parallel with mine.

This algo is called quorum.

quorum: strict majority of total number of nodes accessible(you can talk to).

The downside of quorum is if the network is partitioned into more than 2 sub-graphs(partitions), like the one on the
right of the slide, none of them will keep on working(all of them will be in read-only), because none of them can
identify themselves as the only sub-graph that should keep on working and guarantee that the other ones will shut
themselves down.

### Node failure

Include these categories:

- fail stop failure
- byzantine failure

### Fail stop failure

Easy to deal with.

- checkpoint state and restart strategy: simplest failure is node stop working like when it hits a bug or power outage.
  For this, we can write some code that deals with it. This strategy has a high latency because we wanna restart the
  server/vm.
- fail over to another node strategy(other names: hot spare, replication): for this, we need periodically save your
  state to more than one node(the one that you're currently running on) and you need to have a mechanism like a load
  balancer that can make it so that user reqs can be redirected to the new node when the current node fails.

The second strategy is usually superior. But it costs more.

### byzantine failure

Everything that could go wrong that doesn't cause your node to stop.

To deal with bit flip in memory, we can put checksums on all data structures when you store them into memory and verify
those checksums everytime you read the DS. If checksum verification fails, restart the program.

- Checksums are a bit extreme if bit flop happens a lot, so instead of them, you wanna use more memory and store
  error correcting code instead, so that when you get a checksum validation failure, you can correct the error
  instead of just aborting.
- assertions: put checks and in the checks, make the program die or restart and fallback on whatever strategy you
  implemented already for dealing with **node failure**. So we turned byzantine failure into node stop failure.
- Timeouts: for example if a node gets stuck in a loop, it will perform badly. So we put timeout on every req that we
  send or receive. Now when the node gets stuck, all of it's reqs will get killed and it may restart and comeback in a
  better state.

There's a risk in adopting this strategy. The risk is: If your nodes hit this kind of err too frequently or if they hit
it in a cascading failure across the cluster which requires operator intervention to recover from.

Why needs operator? Because as it happens frequently or in cascading manner, operator needs to prevent it.

### Failure matrix

Failures:

- network failures
- node failures
    - fail stop failures
    - byzantine failures: transform these into fail stop failures, so that we don't have to worry about them

On network side, we use libraries or protocols like ssh and TCP/IP to abstract the network works and treat it as a
series of pipes connecting your nodes. Which packets can either go through or not, and so the conn either works or not.

Once we created the fail stop behavior on the network, you only have to worry about the problem of partitioning of your
network. So our code should do sth sensible(tolerate) the network partition.

## L6: Byzantine Fault Tolerance