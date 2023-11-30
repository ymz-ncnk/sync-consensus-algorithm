# Sync Algorithm
At its core, this algorithm is very similar to Raft, i.e. it is a leader-based 
algorithm that also uses terms (epochs) and stores data in a distribution log.
However, unlike Raft, it synchronizes the distribution log every time a new 
leader appears.

It could be usefull for systems with one-phase write, like 
[this](https://github.com/ymz-ncnk/distributed-key-value-database-design).

## Distributed Log
Each node maintains its own log, which together make a distributed log.

## Become a Leader
Here are some definitions we will use:
- Candidate - a node that lost connection to the leader.
- Manager - a node that has a followers quorum and performs a cluster
  synchronization.
- Leader - a node that can handle user requests.
- Epoch - an integer being always increased for every next leader. It is present 
  on each node, so we can say that a node belongs to a certain epoch.

During the same period of time, several nodes can be candidates, but only one of
them - the node with the largest number of followers - will become the manager.
The main goal of the manager is to synchronize the cluster, i.e. to make sure 
that most nodes have the same data. After that, the manager reliably stores its 
epoch number in the cluster and, if successful, finally becomes the leader.

## Manager Election
The algorithm can be any, and there are no restrictions on the node that can 
become a manager.

## Synchronization
Here are some additional definitions we will use:
- Successful write - when the write request was successfully executed on the 
  nodes quorum.
- Checkpoint - is a log item, all previous items of which have been already
  successfully written.
- Log tail - all log items after the last checkpoint.
- Sync quorum - represents the nodes that participate in the creation of the 
  sync basis.
- Sync basis - is a union of several log tails (starts from the last checkpoint)
  used for synchronization process. It can be represented as a map of 
  intervals `node1: [{1,100}, {105, 200}], node2: {1, 200}, ...`

There are 3 types of preliminary transactions for a new manager:
- Completed - is a transaction that has been executed on all nodes of the sync 
  quorum.
- Interrupted - is a transaction that was executed on several nodes of a sync 
  quorum.
- Interskip (interrupted and skipped) - a transaction that did not get to any 
  node from the sync quorum. It can be identified, for example, by an empty 
  space in the log.

While everything is more or less clear with completed and interskip 
transactions, the processing of interrupted transactions is not so obvious. In 
fact, the new manager does not know whether such a transaction was executed on 
most nodes or not (a few of them may have simply did not get into the sync 
quorum), therefore, it has no choice but to continue its execution. That's why
sync basis is build as a union of the log tails.

Taking all this into account, we can conclude that, a write transaction can 
either be executed or be canceled when a new epoch is entered (i.e. when a new 
leader appears).

### More Details
First of all, we must say, that in a sync quorum, we can have nodes from 
different epochs. But when building the sync basis, we will only be interested 
in the nodes from the most recent epoch, i.e. the nodes with the highest epoch 
number.

Second, a node is synchronized with the sync basis only when it has all 
completed and no interskip transactions. At the same time, nodes from previous 
epochs cannot trust their completed and interrupted transactions, so they have 
to reload the former and delete the latter.

And finally, to speed up the synchronization process, we can resort to some 
optimizations, for example, try to minimize the number of insteskips in the sync 
basis.

## Saving of an Epoch Number
Once the manager has synchronized most of the nodes, it moves on to the next 
stage - reliably stores its epoch number and becomes a leader. This is done in
three steps:
1. First, the manager sends own epoch number to the synchronized nodes as an 
   pre_epoch value.
2. After the pre_epoch is saved on the majority of nodes, it sends to them the 
   same data, but now as an epoch value.
4. After the epoch is saved on the majority of nodes, the manager becomes the
   leader.

But how does the manager know which epoch it belongs to? The manager's epoch 
number is an incremented largest value among the `(pre_epoch, epoch)` pairs 
received from the sync quorum nodes. For example, if it received the following 
pairs from the three nodes:
```
(10, 8)		(8, 9)		(7,8)
```
, its epoch number will equal to 11 (10 + 1).

Thanks to this algorithm, we get a very important property: all nodes 
synchronized by a certain leader will belong to the same, and most importantly, 
unique epoch. That is, there can't be a situation where two nodes from the same 
epoch will have contradictory data.

## And That's It
At this moment, the leader can continue to fill in the log with data again, i.e.
continue interrupted transactions and start new ones.

## Write Transaction
When a leader receives a write request from a client, it starts a write
transaction:
- Makes a transaction ID - its a serial number of the next log item.
- Sends the write request with the transaction ID to all its followers.
- Waits for the write request to be executed on the majority of nodes.
- After that, the write transaction will be considered successful.

## Saving of checkpoints
After the leader has completed several write transactions, it marks the last one
as a checkpoint, which than sends to the followers.

## Left Without Definition
Some details remain undefined:
- Leader election.
- Synchronization process.
