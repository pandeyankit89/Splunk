### Create Inputs in "Splunk DB Connect" App and run every 5 mins :

#### *Input #1:* Active-Queries
```SQL
SELECT pid, usename, datname, state, query_start, query
FROM pg_stat_activity
WHERE state = 'active';
```
#### *Input #2:* All-Client-Connections
```SQL
SELECT pid, usename, client_addr, application_name, backend_start
FROM pg_stat_activity;
```
#### *Input #3:* Availability 
```SQL
SELECT 
	'UP' AS "Database_Status",
	date_trunc('second', now() - pg_postmaster_start_time()) AS "Database_Uptime",
	to_char(pg_postmaster_start_time(), 'YYYY-MM-DD HH24:MI:SS') AS "Database_Started_Time",
	COUNT(*) AS "Used_Connections",
	COUNT(*) FILTER (WHERE state = 'active') AS "Active_Connections",
	COUNT(*) FILTER (WHERE state = 'idle') AS "Idle_Connections",
	COUNT(*) FILTER (WHERE wait_event IS NOT NULL) AS "Waiting_Connections"
FROM 
	pg_stat_activity;
```
#### *Input #4:* Buffer-Clean-Stats 
```SQL
SELECT 
	buffers_clean,
	maxwritten_clean,
	buffers_backend_fsync
FROM 
	pg_stat_bgwriter;
```
#### *Input #5:* Buffer-Writing-Stats
```SQL
SELECT 
	buffers_checkpoint,   -- Buffers written during checkpoints
	buffers_clean,        -- Buffers written by background writer
	buffers_backend,      -- Buffers written by backend processes
	buffers_alloc         -- Buffers allocated
FROM 
	pg_stat_bgwriter;
```
#### *Input #6:* Cache-Hit-Ratio Per Database 
```SQL
SELECT 
	datname,
	blks_hit,
	blks_read,
	ROUND(100 * blks_hit::numeric / NULLIF(blks_hit + blks_read, 0), 2) AS cache_hit_percent
FROM 
	pg_stat_database
ORDER BY 
```
#### *Input #7:* Checkpoint-Stats
```SQL
SELECT 
	checkpoints_timed,
	checkpoints_req,
	checkpoints_timed + checkpoints_req AS total_checkpoints,
	checkpoint_write_time,
	checkpoint_sync_time
FROM 
	pg_stat_bgwriter;
```
#### *Input #8:* Connection-Per-DB
```SQL
SELECT 
	datname,
	numbackends AS connections
FROM 
	pg_stat_database
ORDER BY 
	connections DESC;
```
#### *Input #9:* Databases-Total-Size
```SQL
SELECT 
	d.datname,
	pg_size_pretty(pg_database_size(d.datname)) AS size
FROM 
	pg_database d
ORDER BY 
	pg_database_size(d.datname) DESC
LIMIT 5;
```
#### *Input #10:* DEAD-TUPLES-BLOAT-DETECTION
```SQL
SELECT 
	relname,
	n_live_tup,
	n_dead_tup,
	ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_tuple_pct
FROM 
	pg_stat_user_tables
ORDER BY 
	dead_tuple_pct DESC
LIMIT 10;
```
#### *Input #11:* Frequent-Updated-Tables
```SQL
SELECT 
	schemaname,
	relname,
	n_tup_upd + n_tup_del + n_tup_ins AS total_writes
FROM 
	pg_stat_user_tables
ORDER BY 
	total_writes DESC
LIMIT 10;
```
#### *Input #12:* Idle-Transaction-Connections 
```SQL
SELECT pid, usename, query, state, wait_event_type, wait_event
FROM pg_stat_activity
WHERE state = 'idle';
```
#### *Input #13:* Index-Usage-with-Table-Stats
```SQL
SELECT 
	ui.relname AS table_name,
	ui.indexrelname AS index_name,
	ui.idx_scan,
	ut.seq_scan,
	ROUND(100.0 * ui.idx_scan / NULLIF(ui.idx_scan + ut.seq_scan, 0), 2) AS index_usage_pct
FROM 
	pg_stat_user_indexes ui
JOIN 
	pg_stat_user_tables ut ON ui.relid = ut.relid
ORDER BY 
	index_usage_pct ASC;
```
#### *Input #14:* Largest-Indexes
```SQL
SELECT 
	n.nspname AS schema,
	c.relname AS index_name,
	pg_size_pretty(pg_relation_size(c.oid)) AS index_size
FROM 
	pg_class c
JOIN 
	pg_namespace n ON n.oid = c.relnamespace
WHERE 
	c.relkind = 'i'  -- index
ORDER BY 
	pg_relation_size(c.oid) DESC
LIMIT 10;
```
## *Input #15:* Long-Running-Queries
```SQL
SELECT pid, now() - query_start AS runtime, usename, query
FROM pg_stat_activity
WHERE state = 'active'
AND now() - query_start > interval '5 minutes'
ORDER BY runtime DESC;
```
#### *Input #16:* Most-Used-Indexes
```SQL
SELECT 
	relname AS table_name,
	indexrelname AS index_name,
	idx_scan AS times_used
FROM 
	pg_stat_user_indexes
ORDER BY 
	idx_scan DESC
LIMIT 10;
```
#### *Input #17:* Replication-State
```SQL
SELECT 
	pid,
	usename,
	application_name,
	client_addr,
	backend_start,
	state,
	sync_state
FROM 
	pg_stat_replication;
```
#### *Input #18:* Table-Scan-Type
```SQL
SELECT 
	relname,
	seq_scan,
	idx_scan,
	ROUND(100.0 * seq_scan / NULLIF(seq_scan + idx_scan, 0), 2) AS seq_scan_ratio
FROM 
	pg_stat_user_tables
ORDER BY 
	seq_scan_ratio DESC;
```
#### *Input #19:* Temp-File-And-Bytes
```SQL
SELECT 
	datname,
	temp_files,
	pg_size_pretty(temp_bytes) AS temp_bytes
FROM 
	pg_stat_database
ORDER BY 
	temp_bytes DESC;
```
#### *Input #20:* Transaction-Commited-&-Rollbacks
```SQL
SELECT 
	datname,
	deadlocks,
	xact_commit,
	xact_rollback,
	(xact_rollback::numeric / NULLIF(xact_commit + xact_rollback, 0)) * 100 AS rollback_percent
FROM 
	pg_stat_database
ORDER BY 
	rollback_percent DESC;
```
#### *Input #21:* Unused-Indexes
```SQL
SELECT 
	schemaname,
	relname AS table_name,
	indexrelname AS index_name,
	idx_scan
FROM 
	pg_stat_user_indexes
WHERE 
	idx_scan = 0
ORDER BY 
	table_name;
```

#### *Input #22:* Unused_or_Stale_Indexes
```SQL
SELECT
	s.schemaname,
	s.relname AS table_name,
	s.indexrelname AS index_name,
	idx.indisunique AS is_unique,
	idx.indisprimary AS is_primary,
	s.idx_scan,
	pg_size_pretty(pg_relation_size(s.indexrelid)) AS index_size
FROM
	pg_stat_user_indexes s
JOIN
	pg_index idx ON idx.indexrelid = s.indexrelid
JOIN
	pg_class i ON i.oid = s.indexrelid
WHERE
	s.idx_scan = 0
	AND NOT idx.indisprimary
	AND NOT idx.indisunique
ORDER BY
	pg_relation_size(s.indexrelid) DESC;

```
#### *Input #23:* Vaccum_Analyze_History
```SQL
SELECT 
	relname,
	last_vacuum,
	last_autovacuum,
	last_analyze,
	last_autoanalyze
FROM 
	pg_stat_user_tables
ORDER BY 
	last_autovacuum NULLS FIRST;

```
---

