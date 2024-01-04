# Syncfull Consensus Algorithm
Syncfull consensus algorithm is an extension of the 
[Sync](https://github.com/ymz-ncnk/sync-consensus-algorithm/blob/main/Sync%20Consensus%20Algorithm.md) 
alorithm. It assumes the existence of a state machine on each node in addition 
to the log and states the following:
- After receiving a checkpoint from the manager/leader, the follower should 
  apply all previous unapplied log items to the state machine.
- The manager/leader sends a checkpoint after one or more consecutive successful 
  writes.

## Strong Consistency (Draft)
Strong consistency can be achieved as follows:
- User read requests are processed by only one node - read node. The leader 
  chooses it after synchronization.
- The checkpoint is saved on the read node only after it has been saved on the 
  quorum-1 nodes (otherwise, we will have to complete all interrupted 
  transactions during synchronization). This guarantees the following property:
  the checkpoint of each subsequent read node will always be >= than its 
  predecessor.
- If a read node fails, the leader must wait until the current checkpoint is 
  saved on the quorum nodes, and then select a new read node.