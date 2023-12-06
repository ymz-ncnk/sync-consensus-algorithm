# sync-consensus-algorithm
Hi! 

Not so long ago I became interested in distributed systems, so much that 
developed a new consensus algorithm called Sync (also the 
[distributed key-value database](https://github.com/ymz-ncnk/distributed-key-value-database-design) 
based on it).

In traditional distributed systems, data is first reliably stored in the 
distributed log and only then goes to the state machine. This algorithm is not 
about this (although it can be adapted to such a scenario), it is more about 
systems in which data is stored in the log and state machine simultaneously. 

An example of such a system, can be found in the
[distributed key-value database](https://github.com/ymz-ncnk/distributed-key-value-database-design), 
which is a logical extension of this algorithm and offers a new consistency 
model.

This key-value database, in turn, can be used as a building block for a more 
complex [system](https://github.com/ymz-ncnk/distributed-database-of-aggregates-design) 
that, in addition to storing data, offers search and processing capabilities, 
supports distributed transactions, and can scale to multiple regions.

For now all of this is just an idea, so do not look at it with too serious eyes.
And one more thing, if you have any thoughts or recommendations, I'm open to the
conversation.
