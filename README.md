# sync-consensus-algorithm
In traditional distributed systems, data is first reliably stored in the 
distributed log and only then goes to the state machine. This algorithm is not 
about this (although it can be adapted to such a scenario), it is more about 
systems in which data is stored in the log and state machine simultaneously. An 
example of such a system can be found 
[here](https://github.com/ymz-ncnk/distributed-key-value-database-design).
