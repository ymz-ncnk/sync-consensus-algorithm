# sync-consensus-algorithm
Here you can find two consensus algorithms:
- [Sync](https://github.com/ymz-ncnk/sync-consensus-algorithm/blob/main/Sync%20Consensus%20Algorithm.md) - designed for systems that use only a distributed log. An example of 
  such a system can be found in the [distributed key-value database design](https://github.com/ymz-ncnk/distributed-key-value-database-design), which has 
	some very interesting properties. And, in turn, can be used as a building 
	block for a more complex [system](https://github.com/ymz-ncnk/distributed-database-of-aggregates-design) that, in addition to storing data, offers
	search and processing capabilities, supports distributed transactions, and can
	scale to multiple regions.
- [Syncfull](https://github.com/ymz-ncnk/sync-consensus-algorithm/blob/main/Syncfull%20Consensus%20Algorithm.md) - is an extension of the Sync algorithm, designed for systems that 
  use a state machine in addition to the distributed log.
