---
title: "Database Indexing Strategies for High-Performance Applications"
date: "2024-05-22"
tags: ["database", "performance", "postgresql", "optimization"]
category: "infrastructure"
difficulty: "intermediate"
source: "production-issue"
related_projects: ["codex-app"]
time_spent: "4 hours"
---

# Database Indexing Strategies for High-Performance Applications

## Context

While optimizing the Codex application's search functionality, I encountered significant performance issues with complex queries across multiple content types. This led me to deep dive into database indexing strategies.

## Problem Statement

Our search queries were taking 800ms+ on a dataset of ~10k records. The main issues:

- Full-text search across multiple columns
- Filtering by date ranges and tags
- Sorting by relevance and date

## Solutions Implemented

### 1. Composite Indexes

Created multi-column indexes for common query patterns:

```sql
-- For date-range queries with tag filtering
CREATE INDEX idx_content_date_tags ON content_items (created_at, tags) 
WHERE deleted_at IS NULL;

-- For full-text search with type filtering
CREATE INDEX idx_content_search_type ON content_items 
USING gin(to_tsvector('english', title || ' ' || content), content_type);
```

### 2. Partial Indexes

Focused indexes on active records only:

```sql
-- Only index non-deleted items (90% of queries)
CREATE INDEX idx_active_content ON content_items (created_at) 
WHERE deleted_at IS NULL;
```

### 3. GIN Indexes for JSON

Optimized JSONB tag queries:

```sql
-- For tag-based filtering
CREATE INDEX idx_tags_gin ON content_items USING gin(tags);

-- For metadata searches
CREATE INDEX idx_metadata_gin ON content_items USING gin(metadata);
```

## Performance Results

| Query Type | Before | After | Improvement |
|------------|---------|-------|-------------|
| Text Search | 850ms | 45ms | 94.7% |
| Tag Filter | 320ms | 12ms | 96.2% |
| Date Range | 180ms | 8ms | 95.6% |
| Combined | 1200ms | 25ms | 97.9% |

## Key Learnings

### Index Selection Rules

1. **Most Selective First**: Put the most selective column first in composite indexes
2. **Query Pattern Alignment**: Match index column order to WHERE clause order
3. **Covering Indexes**: Include frequently selected columns to avoid table lookups

### Monitoring Strategies

```sql
-- Find unused indexes
SELECT schemaname, tablename, indexname, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes 
WHERE idx_tup_read = 0;

-- Monitor index usage
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read
FROM pg_stat_user_indexes 
ORDER BY idx_scan DESC;
```

## Trade-offs Discovered

### Benefits
- ✅ Dramatic query performance improvement
- ✅ Better user experience
- ✅ Reduced server load

### Costs
- ❌ Additional storage overhead (~15% increase)
- ❌ Slower write operations (minimal impact)
- ❌ More complex maintenance

## Advanced Techniques Explored

### 1. Expression Indexes

```sql
-- For case-insensitive searches
CREATE INDEX idx_title_lower ON content_items (lower(title));
```

### 2. Index-Only Scans

```sql
-- Include commonly selected columns
CREATE INDEX idx_content_summary ON content_items (id, title, created_at, content_type) 
WHERE deleted_at IS NULL;
```

## Tools and Monitoring

- **EXPLAIN ANALYZE**: Query execution plan analysis
- **pg_stat_statements**: Query performance tracking
- **pgbadger**: Log analysis for slow queries
- **Custom monitoring**: Dashboard for index hit ratios

## Next Steps

- [ ] Implement automated index recommendations
- [ ] Set up alerting for index hit ratio drops
- [ ] Explore materialized views for complex aggregations
- [ ] Consider partitioning for time-series data

## Resources

- [PostgreSQL Documentation - Indexes](https://www.postgresql.org/docs/current/indexes.html)
- [Use The Index, Luke!](https://use-the-index-luke.com/)
- [PostgreSQL Index Types](https://www.postgresql.org/docs/current/indexes-types.html)

## Reflection

This exercise reinforced that performance optimization is about measurement, not assumptions. The dramatic improvements show how crucial proper indexing is for application scalability. The key is balancing query performance with storage and maintenance costs.