# Sync Consensus Algorithm
At its core, the Sync algorithm is very similar to Raft, i.e. it is a 
leader-based algorithm that also uses terms (epochs) and stores data in a 
distributed log. However, unlike Raft, it synchronizes the distributed log 
every time a new leader appears. Another interesting feature it provides is the 
ability to simultaneously write several log items using multiple connections 
(between the leader and the follower), which can significantly increase the 
speed of the distributed system. Thanks to this, we can design a 
[key-value database](https://github.com/ymz-ncnk/distributed-key-value-database-design) that can quickly process both write and read requests. 
Also, the Sync algorithm does not contain such a thing as a state machine. This 
term can be found in the [Syncfull](https://github.com/ymz-ncnk/sync-consensus-algorithm/blob/main/Syncfull%20Consensus%20Algorithm.md) algorithm, which is an extension of Sync.

## Distributed Log
Each node maintains its log, and together they make a distributed log. In the 
following, we will assume that the reader is already familiar with this concept.

## Become a Leader
Here are some definitions we will use:
- Candidate - a node that lost connection to the leader.
- Manager - a node that has a followers quorum and performs a cluster
  synchronization.
- Leader - a node that can handle user requests.
- Epoch - an integer being always increased for every next leader. It is present 
  on each node, so we can say that a node belongs to a certain epoch.

During the same period, several nodes can be candidates, but only one of them 
(the node with the largest number of followers) will become a manager. The 
manager’s main goal is to synchronize the cluster, i.e. to ensure that most 
nodes have the same data. After that, it reliably stores its epoch number in the
cluster and, if successful, finally becomes a leader.

## Manager Election
The algorithm can be any, and there are no restrictions on the node that can 
become a manager.

## Synchronization
Here are some additional definitions we will use:
- Successful write - when the write request was successfully executed on the 
  quorum.
- Checkpoint - is a log item, all previous items of which have been already
  successfully written.
- Log tail - all log items after the last checkpoint.
- Sync quorum - represents the nodes that participate in the creation of the 
  sync basis.
- Sync basis - is a union of several log tails (starting from the largest 
  checkpoint) used for the synchronization process. It can be represented as a 
  map of intervals: `{node1: [{start:1,end:100}, {start:105,end:200}], node2: {start:1,end:200}, ...}`, 
  where each first interval starts with the node's checkpoint.

There are 3 types of preliminary [transactions](#write-transaction) for a new 
manager:
- Completed - is a transaction that has been executed on all nodes of the sync 
  quorum.
- Interrupted - is a transaction that was executed on several nodes of a sync 
  quorum.
- Interskip (interrupted and skipped) - a transaction that not gotten to any 
  node from the sync quorum. It can be identified, for example, by an empty 
  space in the log.

While everything is more or less clear with completed and interskip 
transactions, the processing of interrupted transactions is not so obvious. The 
new manager does not know whether such a transaction was executed on most nodes 
or not (a few of them may have simply not gotten into the sync quorum), 
therefore, it has no choice but to continue its execution. That’s why the sync 
basis is built as a union of the log tails. 

Taking all this into account, we can conclude that a write transaction can 
either be executed or be canceled when a new epoch is entered (i.e. when a new 
leader appears).

### More Details
First of all, it should be noted that the sync quorum can include nodes from 
different epochs. However, when building the sync basis, we will only be 
interested in nodes from the last epoch. Second, a node is synchronized with 
the sync basis only when it has all completed and no interskip transactions. At 
the same time, nodes from previous epochs cannot trust their completed and 
interrupted transactions (that are after the checkpoint), so they have to reload
the former and delete the latter. Finally, to speed up the synchronization 
process, we can resort to some optimizations, for example, try to minimize the 
number of insteskips in the sync basis.

### Synchronization Process
We will call a follower from the last epoch, which has the sync basis checkpoint 
as a ready follower. The ready follower does not need to synchronize if it is in
the sync quorum. Moreover, if the sync quorum is filled only with ready 
followers, the manager can skip the synchronization phase altogether.

To all other followers, the manager sends the sync basis. This allows a node to 
receive data from different sources simultaneously during synchronization, which 
can significantly speed up this process.

#### Dynamic Sync Quorum
Thus, if we have most of the live nodes from the last epoch and they are all in 
the sync quorum, the manager will most likely be able to quickly become a 
leader. But that’s if we’re lucky because many nodes from previous epochs can
randomly get into the sync quorum. In this case, when a ready follower appears, 
the manager can:
1. Add it to the sync quorum instead of one of the not-ready followers.
2. Restart the synchronization process, i.e. re-create the sync basis and send 
   it to all relevant nodes again.

This algorithm can also significantly speed up the synchronization process.

## Saving of an Epoch Number
Once the manager has synchronized most of the nodes, it moves on to the next 
stage — reliably stores its epoch number and becomes a leader. This is done in 
three steps:
1. First, the manager sends its epoch number to the synchronized nodes as a 
   pre_epoch value
2. After the pre_epoch is saved on the majority of nodes, it sends to them the 
   same data, but now as an epoch value.
3. After the epoch is saved on the majority of nodes, the manager becomes a 
   leader.
In general, any node synchronized by the manager/leader will receive a 
(pre_epoch, epoch) pair from it.

But how does the manager know which epoch it belongs to? The manager’s epoch 
number is an incremented largest value among the (pre_epoch, epoch) pairs 
received from the sync quorum nodes. For example, its epoch number will equal 
11 if it receives the following pairs: (10, 8), (8, 9), (7,8). Thanks to this 
algorithm, we get a very important property: all nodes synchronized by a certain
leader will belong to the same, and most importantly, unique epoch. That is, 
there can’t be a situation where two nodes from the same epoch will have 
contradictory data.

## And That's It
At this moment, the leader can continue to fill in the log with data again, i.e.
continue interrupted transactions and start new ones.

## Write Transaction
When the leader receives a write request from a client, it starts a write 
transaction:
- Makes a transaction ID — a serial number of the corresponding log item.
- Sends a write request with the transaction ID to all its followers.
- Waits for the write request to be executed on the majority of nodes.
- After that, the write transaction will be considered successful.

## Saving of Checkpoints
After the leader has completed several write transactions, it marks the last one
as a checkpoint, which is then sent to the followers.

## Sync Algorithm Properties
Summarizing, here are some of the main Sync algorithm properties:
- Log items can be written to nodes in any order.
- The log may contain empty spaces.
- Data is considered reliably stored if it is present on the majority of nodes.
- The log is synchronized every time the leader changes.
- If we have a quorum of nodes from the previous epoch, synchronization will 
  take a short period of time.

## Simultaneous Write to the Log
For example, let’s describe the structure of a distributed log that supports the
simultaneous writing of several items. On each node, the log will consist of two
files located on different hard disks (i.e., they can be written to in 
parallel). The first file will contain every first log item received from the 
first connection to the leader, and the second file will contain every second 
log item received from the second connection. The Sync algorithm can ensure the 
operation of such a distributed log, despite the possible unpredictable order in
which its elements are saved.