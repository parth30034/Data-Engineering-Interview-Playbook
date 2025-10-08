🧱 Part 1 — Azure Data Factory (ADF) Data Ingestion & Optimization Playbook
===========================================================================

🎯 **Goal:** Build a production-ready understanding from **setup → ingestion → optimization → best practices**.

**1️⃣ Foundation Setup — Building the ADF Backbone**
----------------------------------------------------

### 🔧 A. Linked Services

Linked Services are the “connection definitions” ADF uses to connect to data sources/destinations.

**Common Types:**

SourceLinked Service TypeKey ParametersAzure Blob / ADLS Gen2Azure Blob StorageStorage account name, Key/Managed IdentityAzure SQL DB / SynapseAzure SQL DatabaseServer name, DB name, Auth methodOn-Prem SQL ServerSelf-Hosted IR + SQL ServerConnection string, IR nameREST / APIHTTPURL, Headers, Auth (OAuth, Basic)

**Best Practices:**

*   Use **Managed Identity or Key Vault Secrets** instead of keys.
    
*   Use **Parameterization** for dynamic environments (dev/test/prod).
    
*   LS\_ADLS\_DEV LS\_SQL\_PROD
    

### ⚙️ B. Integration Runtime (IR)

It’s the **compute engine** that moves and transforms data.

TypeUse Case**Azure IR**For cloud-to-cloud copy**Self-Hosted IR**For on-prem → cloud**SSIS IR**For legacy SSIS package lift & shift

**Optimization Tips:**

*   For large data, scale up **IR core count (4 → 32)**.
    
*   Use **region proximity** (IR near data source).
    
*   Avoid cross-region data movement.
    

### 🧩 C. Datasets & Parameters

Datasets define _what data to move_. Parameterization allows reusability.

Example:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   {    "name": "DS_ADLS_Generic",    "type": "Dataset",    "properties": {      "type": "DelimitedText",      "parameters": { "folderName": "", "fileName": "" },      "linkedServiceName": { "referenceName": "LS_ADLS", "type": "LinkedServiceReference" },      "typeProperties": {        "location": { "type": "AzureBlobStorageLocation", "folderPath": "@{dataset().folderName}" },        "fileName": "@{dataset().fileName}"      }    }  }   `

👉 Now your dataset can be reused for any folder/file combination.

**2️⃣ Data Ingestion Scenarios — Real World Design Patterns**
-------------------------------------------------------------

### 🧱 A. Full Load / Batch Ingestion

**Use Case:** Static or low-frequency data (reference tables, configs).

**Pipeline Design:**

*   Copy Activity: SQL → ADLS (CSV/Parquet)
    
*   Trigger: Manual or scheduled
    
*   Output Path: /bronze/full\_load/yyyy=2025/mm=10/dd=08/
    

**Optimization:**

*   Enable **staged copy** (SQL → Blob staging → ADLS).
    
*   Enable **parallel copy** for multiple tables.
    

### 🔁 B. Incremental Load (Watermark / Delta)

**Use Case:** Tables with LastModifiedDate or UpdatedAt.

**Pattern:**

1.  Lookup Activity → fetch last watermark from metadata table
    
2.  Copy Activity → use dynamic query
    
3.  Update metadata table with new max value
    

**Dynamic Query Example:**

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   SELECT * FROM dbo.Orders  WHERE UpdatedAt > @{pipeline().parameters.LastWatermark}   `

**Optimization:**

*   Use **partitioned queries** (SQL hint: OPTION (QUERYTRACEON 8649) for parallelism)
    
*   Limit copy concurrency based on target I/O bandwidth.
    

### 🧬 C. Change Data Capture (CDC)

**Approaches:**

1.  Use **Azure SQL CDC tables** → filter by \_\_$operation flags.
    
2.  Or use **Databricks Delta CDC merge** downstream (simpler).
    

**Pipeline Design:**

*   Lookup → CDC-enabled SQL View
    
*   Copy Activity → ADLS /bronze/cdc/
    
*   Tag record with operation\_flag
    

### ⚡ D. Near Real-Time / Event-based Ingestion

**Event-Driven Architecture using:**

*   Event Grid → triggers pipeline on file upload.
    
*   Logic App / Function → triggers ADF REST API.
    
*   ADF → calls Synapse/Databricks for immediate transformation.
    

**Example:**

> New file lands in ADLS → Event Grid triggers ADF → Pipeline copies → triggers Databricks notebook.

### 🧮 E. Metadata-Driven Dynamic Pipelines

**Goal:** Single pipeline handles multiple sources.

*   source\_tablesource\_querytarget\_pathload\_typecustomerSELECT \* FROM .../adls/customerincremental
    
*   ADF reads metadata → executes **ForEach Activity** → calls Copy Activity dynamically.
    

**Benefits:**

*   One pipeline replaces 100s.
    
*   Easier maintenance & CI/CD.
    

**3️⃣ Optimization Techniques — ADF in Production**
---------------------------------------------------

### ⚙️ Copy Activity

*   Use **PolyBase** or **Bulk Copy (BCP)** when ingesting to Synapse.
    
*   For large files, enable **compression (gzip/snappy)** to reduce network cost.
    
*   "parallelCopies": 8
    
*   Monitor through **ADF Monitor + Log Analytics**.
    

### ⚙️ Data Flow Activity

*   For transformations (light only):
    
    *   Enable **Data Flow Debug** for dev only (disable in prod).
        
    *   Use **partitioning strategy** (key, hash, round robin).
        
    *   Cache small dimension datasets.
        

### ⚙️ Pipeline Architecture

*   Avoid nesting too deep (max 2-3 levels of Execute Pipeline).
    
*   Use **Lookup + ForEach + Execute Pipeline** pattern carefully; prefer **Batch Grouping**.
    
*   Implement **Error Handling**:
    
    *   OnFailure path → Notification / Log pipeline.
        

**4️⃣ Monitoring, Logging & Alerts**
------------------------------------

### 🧩 Enable:

*   Diagnostic logs → Log Analytics workspace.
    
*   Pipeline-level custom logging → Azure SQL table.
    
*   Alerts → via Azure Monitor (pipeline failure, runtime threshold).
    

**Example:**

> If duration > 30 min or rows < expected, trigger alert.

**5️⃣ Industry Best Practices (ADF)**
-------------------------------------

✅ **Design Patterns:**

*   Use **Config + Metadata Driven** pipelines
    
*   Store **Secrets in Key Vault**
    
*   Reuse **Linked Services / Datasets via Parameters**
    
*   Implement **Retry Logic** (3 attempts max)
    
*   Enable **Logging & Audit columns**
    

🚫 **Anti-Patterns:**

*   Hardcoding file paths or connections
    
*   Performing heavy transformations in ADF Data Flows (use Synapse/Databricks instead)
    
*   Ignoring IR region placement
    
*   Single-threaded ForEach without concurrency
    

**6️⃣ Real-World Example: Metadata-Driven Incremental Ingestion**
-----------------------------------------------------------------

**Architecture Flow:**

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   SQL Server → ADF Pipeline → ADLS Gen2 (Bronze)   `

**Steps:**

1.  Lookup → read metadata
    
2.  ForEach → iterate tables
    
3.  Copy → dynamic query with watermark
    
4.  Update watermark in SQL table
    
5.  Trigger downstream Databricks job (Silver)
    

**Mermaid Diagram:**

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   graph TD  A[Metadata Table] --> B[Lookup Activity]  B --> C[ForEach Table]  C --> D[Copy Activity]  D --> E[ADLS Bronze Layer]  E --> F[Update Watermark]  F --> G[Trigger Databricks Job]   `

**7️⃣ Advanced Optimization Checklist**
---------------------------------------

CategoryOptimization**Copy Data**Enable parallel copies, staging, compression**Data Flows**Use partitioning, reduce joins, disable debug**Integration Runtime**Scale up, region proximity, auto-resolve**Pipelines**Retry, logging, concurrency, batch triggers**Cost**Use serverless IR when idle; turn off debug**Security**Managed identities, Key Vault, RBAC

✅ Deliverables Summary
----------------------

You now have:

*   🧱 Foundation Setup (connections, IR, datasets)
    
*   🔁 Ingestion Scenarios (batch, incremental, CDC, streaming)
    
*   ⚙️ Optimization (performance, cost, IR scaling)
    
*   🧠 Best Practices (reusability, security, monitoring)
    
*   🧩 Example architecture + checklist