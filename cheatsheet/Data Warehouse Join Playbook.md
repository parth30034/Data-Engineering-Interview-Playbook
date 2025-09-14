# 📘 Data Warehouse Join Playbook (Redshift + Synapse)

## 🎯 Goal
Efficiently join large fact tables with dimensions in Amazon Redshift and Azure Synapse, avoiding costly data movement and skew.

## 🛠️ Common Join Challenges
- **Data movement / shuffle** → when join keys aren't collocated
- **Skew** → hot key overloads a single node
- **Wrong distribution** → fact and dim distributed differently
- **Oversized broadcast** → replicating "small" tables that aren't really small
- **Stale stats** → poor join strategy chosen by optimizer
- **Too many joins** → deep snowflake schemas slow BI queries

## 🟢 Amazon Redshift — Best Practices
- **DISTKEY**: Align fact and frequently-joined dim on the same DISTKEY
- **DISTSTYLE ALL**: Replicate tiny, static dims (date, geo)
- **SORTKEY**: Use on join + filter columns for efficient merge joins
- **ANALYZE & VACUUM**: Keep stats fresh for optimizer
- **Spectrum Joins**: Partition + columnar (Parquet, 128MB+ files)

### 📌 Example: Joining `fact_ad_events` with `dim_campaign`
- Make `campaign_id` the DISTKEY
- Use compound SORTKEY (`campaign_id`, `event_date`)
- Replicate `dim_date`

## 🔵 Azure Synapse — Best Practices
- **Distribution**:
  - HASH fact and large dim on same key (e.g., `campaign_id`)
  - REPLICATE small dims (date, region)
  - Avoid ROUND_ROBIN unless staging
- **Columnstore partitions**: Ensure ~1M+ rows per distribution/partition for efficiency
- **CTAS**: Re-distribute table before heavy joins
- **Update Stats**: Keep statistics current for correct join strategy

### 📌 Example: Joining `fact_ad_events` with `dim_campaign`
- Hash distribute both on `campaign_id`
- Replicate `dim_date`
- Partition by `event_date` for time-based queries

## 🚨 Diagnostics Checklist
- Run `EXPLAIN` — check for "redistribute" or "broadcast" steps
- Check row skew per distribution — any one node holding >2× average rows?
- Confirm matching datatypes on join keys
- Validate table stats (`ANALYZE` / `UPDATE STATISTICS`)
- For external tables — ensure large Parquet files (64–256 MB), partitioned

## 📈 Universal Mitigations
- Pre-aggregate into marts (daily/hourly aggregates)
- Use materialized views for dashboards
- Denormalize where feasible — avoid excessive joins
- Monitor query runtime, WLM (Redshift) or DMVs (Synapse) for data movement
- Salt/skew-handling techniques for hot keys

## 🧩 Interview Soundbite
> "In large-scale warehouses, joins are slow mainly due to data movement and skew. In Redshift I control this with DISTKEY + SORTKEY and replication of small dims. In Synapse I use HASH distribution on the join key or replication. In both, I always check EXPLAIN for redistribution steps and fix by aligning keys or building marts. For dashboards, I rely on pre-aggregated marts instead of hitting raw facts."

## ✅ Summary
That's your one-page reference — covers use case, challenges, platform strategies, diagnostics, and interview phrasing.
