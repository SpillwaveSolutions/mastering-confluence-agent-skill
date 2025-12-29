# Atlassian MCP Tools Reference

Reference for Confluence operations via `mcp__atlassian` server.

## Important Limitation

**DO NOT use MCP for uploading large pages** - size limits (~10-20KB) cause failures.

Use MCP for: Reading, searching, small updates
Use REST API scripts for: Large uploads, pages with images

## Available Tools

### confluence_search

Search pages using CQL.

```javascript
mcp__atlassian__confluence_search({
  query: 'space = "DEV" AND text ~ "API"',
  limit: 25
})
```

**Parameters:**
- `query` (required): CQL query string
- `limit`: Max results (default: 25)
- `start`: Offset for pagination

**Returns:** Array of page objects with id, title, space, URL

### confluence_get_page

Retrieve page content.

```javascript
mcp__atlassian__confluence_get_page({
  page_id: "780369923",
  include_metadata: true
})
```

**Parameters:**
- `page_id` (required): Page ID
- `include_metadata`: Include version, labels, etc.

**Returns:** Page object with content, metadata

### confluence_create_page

Create new page (small content only).

```javascript
mcp__atlassian__confluence_create_page({
  space_key: "DEV",
  title: "New Page",
  content: "<p>Content here</p>",
  content_format: "storage",
  parent_id: "123456"
})
```

**Parameters:**
- `space_key` (required): Space key
- `title` (required): Page title
- `content` (required): Page content
- `content_format`: "storage" (HTML) or "wiki"
- `parent_id`: Optional parent page ID

### confluence_update_page

Update existing page (small content only).

```javascript
mcp__atlassian__confluence_update_page({
  page_id: "780369923",
  title: "Updated Title",
  content: "<p>New content</p>",
  content_format: "storage",
  version_comment: "Updated via MCP"
})
```

**Parameters:**
- `page_id` (required): Page ID
- `title`: New title (optional)
- `content`: New content
- `content_format`: "storage" or "wiki"
- `version_comment`: Commit message

### confluence_add_label

Add label to page.

```javascript
mcp__atlassian__confluence_add_label({
  page_id: "780369923",
  label: "api"
})
```

### confluence_get_labels

Get page labels.

```javascript
mcp__atlassian__confluence_get_labels({
  page_id: "780369923"
})
```

### confluence_get_page_children

Get child pages.

```javascript
mcp__atlassian__confluence_get_page_children({
  page_id: "780369923"
})
```

## When to Use MCP vs Scripts

| Task | Use MCP | Use Script |
|------|---------|------------|
| Search pages | Yes | - |
| Read page content | Yes | - |
| Small page create (<10KB) | Yes | - |
| Large page upload | No | upload_confluence_v2.py |
| Page with images | No | upload_confluence_v2.py |
| Download with attachments | No | download_confluence.py |
| Add/get labels | Yes | - |
| Get children | Yes | - |

## Error Handling

| Error | Meaning | Solution |
|-------|---------|----------|
| `401 Unauthorized` | Invalid credentials | Check MCP server config |
| `404 Not Found` | Page doesn't exist | Verify page_id |
| `413 Payload Too Large` | Content too big | Use REST API script |
| `Rate Limited` | Too many requests | Add delays between calls |

## Combining with Scripts

Common pattern: Use MCP to find pages, scripts to update:

```python
# 1. Search with MCP to find page ID
results = mcp__atlassian__confluence_search(query='...')
page_id = results[0]['id']

# 2. Use script for large upload
# python3 scripts/upload_confluence_v2.py doc.md --id {page_id}
```

See [troubleshooting_guide](troubleshooting_guide.md) for detailed error solutions.
