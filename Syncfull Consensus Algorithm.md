# Syncfull Consensus Algorithm
Syncfull consensus algorithm is an extension of the 
[Sync](https://github.com/ymz-ncnk/sync-consensus-algorithm/blob/main/Sync%20Consensus%20Algorithm.md) 
alorithm. It assumes the existence of a state machine on each node in addition 
to the log and states the following:
- After receiving a checkpoint from the manager/leader, the follower should 
  apply all previous unapplied log items to the state machine.
- Manager/leader sends a checkpoint after one or more consecutive successful 
  writes.
	