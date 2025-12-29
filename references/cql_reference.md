# CQL Reference

Confluence Query Language (CQL) syntax for searching pages.

## Basic Syntax

```sql
field operator value [AND|OR field operator value]
```

## Common Fields

| Field | Description | Example |
|-------|-------------|---------|
| `space` | Space key | `space = "DEV"` |
| `title` | Page title | `title ~ "API"` |
| `text` | Content text | `text ~ "authentication"` |
| `type` | Content type | `type = page` |
| `label` | Label/tag | `label = "api"` |
| `creator` | Author | `creator = currentUser()` |
| `created` | Creation date | `created >= startOfYear()` |
| `lastModified` | Last edit date | `lastModified >= now("-7d")` |
| `ancestor` | Parent in hierarchy | `ancestor = 123456` |
| `parent` | Direct parent | `parent = 123456` |

## Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `=` | Exact match | `space = "DEV"` |
| `!=` | Not equal | `type != attachment` |
| `~` | Contains (text search) | `text ~ "error"` |
| `!~` | Does not contain | `title !~ "draft"` |
| `IN` | Multiple values | `label IN ("api", "docs")` |
| `NOT IN` | Excludes values | `space NOT IN ("TEST", "SANDBOX")` |
| `>=`, `<=`, `>`, `<` | Date comparisons | `created >= "2024-01-01"` |

## Date Functions

| Function | Description |
|----------|-------------|
| `now()` | Current time |
| `now("-7d")` | 7 days ago |
| `now("-1M")` | 1 month ago |
| `startOfDay()` | Start of today |
| `startOfWeek()` | Start of this week |
| `startOfMonth()` | Start of this month |
| `startOfYear()` | Start of this year |

## Example Queries

### Search by Space

```sql
space = "DEV" AND type = page
```

### Text Search

```sql
text ~ "authentication" AND space = "API"
```

### Recent Changes

```sql
space = "DEV" AND lastModified >= now("-7d") ORDER BY lastModified DESC
```

### By Label

```sql
label IN ("api", "documentation") AND space = "DEV"
```

### By Creator

```sql
creator = currentUser() AND created >= startOfMonth()
```

### Complex Query

```sql
space = "DEV"
AND type = page
AND (label = "api" OR label = "documentation")
AND created >= "2024-01-01"
AND text ~ "authentication"
ORDER BY lastModified DESC
```

### Exclude Archived

```sql
space = "DEV" AND type = page AND label NOT IN ("archived", "deprecated")
```

## Using with MCP

```javascript
mcp__atlassian__confluence_search({
  query: 'space = "DEV" AND text ~ "API" ORDER BY created DESC',
  limit: 25
})
```

## Sorting

```sql
ORDER BY created ASC     # Oldest first
ORDER BY created DESC    # Newest first
ORDER BY lastModified DESC
ORDER BY title ASC
```

## Pagination

Results are paginated. Use `start` and `limit` parameters:

```javascript
confluence_search({ query: '...', limit: 25, start: 0 })  // First 25
confluence_search({ query: '...', limit: 25, start: 25 }) // Next 25
```

## Escaping

- Quotes in values: Use backslash `\"` or single quotes
- Special characters: Escape with backslash

```sql
title ~ "User's Guide"
text ~ "error: \"not found\""
```
