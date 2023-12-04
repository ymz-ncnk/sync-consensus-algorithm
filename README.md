# sync-consensus-algorithm
Hi! 

Not so long ago I became interested in distributed systems, so much that 
invented, as for me, pretty interesting consensus algorithm called Sync (also 
the [distributed key-value database design](https://github.com/ymz-ncnk/distributed-key-value-database-design) 
based on it). And I would be very interested to hear comments or recommendations 
from more experienced people in this area. But please do not look at it with too 
serious eyes, at this stage it is just an idea.

In traditional distributed systems, data is first reliably stored in the 
distributed log and only then goes to the state machine. This algorithm is not 
about this (although it can be adapted to such a scenario), it is more about 
systems in which data is stored in the log and state machine simultaneously. 
