## Architecture

```
S3 Bucket (10 MB sample data)
    ↓
Glue Crawler (1 run, 2 DPUs, 2 minutes)
    ↓
Glue Data Catalog (3 tables)
    ↓
Lake Formation (Permissions management)
    ↓
Athena (Query with 100 MB scan limit)
```

---

## Learning Objectives

Students will learn:
✅ Lake Formation data lake registration
✅ Grant/revoke table permissions
✅ Column-level security
✅ Row-level filtering (basic)
✅ Query data with Athena
✅ Audit data access

**NOT included (to save costs):**
❌ Glue ETL jobs
❌ Large dataset processing
❌ Continuous crawlers
❌ Cross-account sharing
❌ EMR/Redshift Spectrum integration

---

## Workshop Duration

- **Setup:** 15 minutes (instructor)
- **Hands-on:** 90 minutes (students)
- **Cleanup:** 15 minutes (automated)
- **Total:** 2 hours

---

### Exercise 1: Register Data Lake (10 min)
- Register S3 bucket with Lake Formation
- Verify data lake administrator permissions

### Exercise 2: Run Glue Crawler (15 min)
- Create Glue database
- Configure crawler for sample data
- Run crawler (1 time only)
- Verify tables created

### Exercise 3: Grant Permissions (20 min)
- Create Lake Formation permissions
- Grant SELECT on tables
- Test permissions with Athena

### Exercise 4: Column-Level Security (20 min)
- Create filtered table with column exclusions
- Grant permissions with column filters
- Query and verify PII columns hidden

### Exercise 5: Row-Level Filtering (25 min)
- Create data filter for specific rows
- Apply filter to table permissions
- Query and verify row filtering works

---
