# AWS Scenario Q&A Cheatsheet 

Structured answers using **Problem → Constraints → Choice → Why → Alternatives**.

---

## Scenario 1: Batch ingestion from S3 → analytics
**Q:** You have large CSV files arriving daily into S3. Analysts need this data in an analytics layer for querying. Which AWS service(s) would you use and why?

**Answer:**
- **Problem:** Need to ingest structured batch data from S3 into analytics-ready format.
- **Constraints:** Large files, daily arrival, analysts need SQL queries, cost matters.
- **Choice:** Use **AWS Glue** for ETL, store in **Parquet/Delta on S3**, and expose via **Athena** or load into **Redshift** for frequent queries.
- **Why:** Glue handles schema discovery + ETL, Parquet/Delta reduces cost and improves query performance. Redshift provides low-latency joins and aggregations.
- **Alternatives:** Athena for ad-hoc/low-frequency queries, EMR Spark for very large/complex transforms.

**Follow-up prompt:** When would you choose Redshift vs Athena?
- Redshift: frequent queries, joins, dashboards.
- Athena: ad-hoc queries, low engineering overhead, cost-per-query model.

---

## Scenario 2: Schema evolution in S3 (Parquet/Delta)
**Q:** You’re storing event data in S3 as Parquet. The schema evolves over time (new columns, renamed columns). How would you handle this?

**Answer:**
- **Problem:** Schema changes break downstream queries if unmanaged.
- **Constraints:** Backward compatibility, no data loss, easy maintenance.
- **Choice:** Use **Delta Lake on S3** for schema evolution. Delta supports `MERGE`, time travel, schema enforcement/evolution (`mergeSchema`).
- **Why:** Delta gives ACID guarantees, allows new columns while keeping old data accessible, simplifies downstream queries.
- **Alternatives:** Glue catalog with Parquet (needs schema registry/versioning), JSON for flexibility (but inefficient).

**Follow-up prompt:** What if a column’s data type changes (string → int)?
- Requires explicit ETL migration (cast + backfill). Schema evolution handles adds/removes, but type changes need controlled migration.

---

## Scenario 3: Orchestration — Step Functions vs Airflow vs Glue Workflows
**Q:** You need to orchestrate a multi-step ETL pipeline in AWS. Which service would you pick and why?

**Answer:**
- **Problem:** Orchestrate dependent ETL tasks.
- **Constraints:** Mix of AWS-native jobs (Glue/S3/Lambda) + Spark jobs. Need retries, monitoring, alerts.
- **Choice:** **Step Functions** for AWS-native jobs. If multi-cloud or advanced DAGs, use **MWAA (Managed Airflow)**.
- **Why:** Step Functions = AWS-native, serverless, less ops. Airflow = flexibility, plugins, complex DAGs.
- **Alternatives:** Glue Workflows (simple, if only Glue jobs). Self-managed Airflow (on EC2/EKS) for heavy customization.

**Follow-up prompt:** When prefer Step Functions vs Airflow?
- Step Functions: small/medium, event-driven, AWS-native pipelines.
- Airflow: complex DAGs, cross-cloud, advanced customization.

---

### Interview delivery tips
- Structure answers in 90–120s using **Problem → Constraints → Choice → Why → Alternatives**.
- End with trade-offs and when you’d use alternatives.
- Keep examples grounded in data engineering (S3, Glue, Delta, Spark, Redshift).

