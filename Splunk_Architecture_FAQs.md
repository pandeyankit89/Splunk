### Splunk Architecture FAQs :
---
#### (1) How Data flow from UF/HF to Indexer :
```
[Universal/Heavy Forwarder]
          |
          | (Data via splunktcp on 9997)
          v
[     Indexer       ]
		  ├── Parsing Queue (line-breaking, timestamping)
		  └── Indexing Queue
					|
					|			
					v
[ Index Location : /opt/splunk/var/lib/splunk/<index_name> ]
					|			
					v
[ Bucket Location : Hot,Warm = <index_name>/db  Cold = <index_name>/colddb directory> ]
					|			
					v
[ Bucket : /db_<latest_epoch_time>_<earliest_epoch_time>_<bucket_id> ]
		├── rawdata/journal.gz   (compressed raw events)
		├── *.tsidx              (index files for search)
		├── Metadata files       (state & lookup info)
```
---

#### (2) What is Splunk Indexer Location ?
- /opt/splunk/var/lib/splunk/<index_name>
---

#### (3) What is the location of buckets ?
- Hot and Warm buckets = /opt/splunk/var/lib/splunk/<index_name>/*_db/_*  
- Cold buckets = <index_name>/*_colddb/_*
- Thawed buckets = <index_name>/*_thawed/_*
---

#### (4) What is the bucket naming convention ?
- db_<latest_epoch_time>_<earliest_epoch_time>_<bucket_id>

---
#### (5) What is the difference between Replication-Factor of Indexer Cluster and Search-Head Cluster? What does each cluster replicate?
- **Indexer Cluster**: Replication-Factor = number of copies of raw data/ data-buckets across indexers for high availability.  
- **Search-Head Cluster**: Replicates search artifacts (knowledge bundles, search jobs) for high availability.

---

#### (6) What is a Search Artifact?
Search artifacts are _temporary results and metadata_ generated during a search, like dispatch directory, *search results*, and job logs.

---

#### (7) What are rawdata.journal and tsidx files? What is their size and when/where are they created?
- **rawdata/journal.gz**: is the _pre-indexed_, compressed data file (around 15% of actaul input).  
- **tsidx**: time-series _index_ files for fast search. Created when data is indexed inside buckets. Size depends on data volume and compression.

---

#### (8) What happens if the Index Cluster Manager is down but other indexers in the cluster are still running?
Indexing and searching continue with existing data, but cluster won’t manage replication or fix bucket issues until manager is restored. _Peer-to-peer communication for replication and fixup activities will be paused_ until the manager returns.

---

#### (9) What happens if the Index Cluster Manager is down and one indexer in the cluster is also down, while remaining indexers are running?
Cluster won’t rebalance or replicate missing copies, risking data availability if more indexers fail.

---

#### (10) What happens if the Index Cluster Manager is down and one indexer (which was down) comes back up?
Indexer rejoins but cluster manager can’t validate or fix buckets, so data consistency may be impacted until manager returns.

---

#### (11) What happens if the Index Cluster Manager (which was down) comes back up while other indexers are running?
Manager resumes coordination, triggers bucket fix-ups and ensures replication/search-factor compliance.

---

#### (12) If I have *n* indexers, what is the best practice to set the Search-Factor and Replication-Factor?
- Replication Factor = at least 3 (for durability).  
- Search Factor = Replication Factor – 1 (for balanced performance vs storage).
- For 'n' indexers, ensure RF + SF <= n to avoid single points of failure.
---

#### (13) Why is it a best practice to keep the Search-Factor one less than Replication-Factor in an Indexer Cluster?
Because not all replicated copies need to be searchable. This saves storage and CPU while ensuring enough searchable copies for redundancy.

---

#### (14) What happens if the Deployer goes down while other Search-Heads are still running?
Nothing immediately—search-heads continue running. Deployer is only used for pushing configuration/apps. You simply cannot push new app configurations or bundle updates until the Deployer is restored.

---

#### (15) What happens if the Deployer (which was down) comes back up while other Search-Heads are still running?
It simply becomes available again to push future configs. _No automatic sync happens._ The Deployer will push its apps _to the captain_, which will then redistribute the bundle to all members.

---

#### (16) What happens if a Search-Head goes down in a Search-Head Cluster while the Captain and other members are still running?
Cluster continues functioning, searches redistribute. The downed SH will resync upon return.

---

#### (17) What happens if the Captain goes down in a Search-Head Cluster and other search-heads are running?
Remaining search-heads hold an election, and a new Captain is chosen automatically.

---

#### (18) How does a Search-Head Cluster decide a Captain?
Through an automatic election process based on majority consensus and cluster state.

---

#### (19) Can we make a specific Search-Head the Captain if the Captain is down and only 2 Search-Heads are running?
No, you cannot force it manually. With 2 members, quorum is weak, but one will still be elected automatically. You can influence it by setting a member's priority, but you cannot manually force a specific member to become captain.

---

#### (20) What happens if the Captain (which went down) comes back up while other Search-Heads are running?
It rejoins as a regular (non-captain) member. The current Captain remains unless another election is triggered.

---
