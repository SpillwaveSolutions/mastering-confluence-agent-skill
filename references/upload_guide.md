# Upload Guide

Complete guide for uploading Markdown content to Confluence.

## Script Reference

**Script:** `scripts/upload_confluence_v2.py`

```bash
# Basic usage
python3 scripts/upload_confluence_v2.py document.md --id PAGE_ID

# Options
--id PAGE_ID        # Page ID for updates (required if not creating)
--space SPACE_KEY   # Space key for new pages
--parent-id ID      # Parent page ID for hierarchy
--title "Title"     # Override title (defaults to H1 or frontmatter)
--dry-run           # Preview without uploading
--force-reupload    # Re-upload existing attachments
--env-file PATH     # Custom credentials file
```

## Workflow Steps

### 1. Prepare Content

Ensure markdown follows these rules:
- Images use markdown syntax: `![alt](path)`
- No raw Confluence XML in content
- Diagrams converted to PNG/SVG (not Mermaid/PlantUML code blocks)

### 2. Verify Images Exist

```bash
# List images referenced in markdown
grep -o '!\[.*\]([^)]*\.png)' document.md

# Verify files exist
ls -lh ./diagrams/
```

### 3. Test with Dry-Run

```bash
python3 scripts/upload_confluence_v2.py document.md --id PAGE_ID --dry-run
```

Check output for:
- Correct mode (UPDATE/CREATE)
- All attachments found
- Content preview looks correct

### 4. Execute Upload

```bash
python3 scripts/upload_confluence_v2.py document.md --id PAGE_ID
```

### 5. Verify Result

Open the returned URL and confirm:
- Content renders correctly
- Images display properly
- Links work

## Frontmatter Support

The script reads YAML frontmatter:

```yaml
---
title: Page Title
confluence:
  id: "780369923"
  space: "DEV"
parent:
  id: "123456"
---
```

CLI flags override frontmatter values.

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| `401 Unauthorized` | Invalid credentials | Check .env file |
| `404 Not Found` | Wrong page ID | Verify ID from URL |
| `Version conflict` | Concurrent edit | Retry (script fetches latest) |
| `File not found` | Image path wrong | Use absolute paths or verify relative |

## API Details

Uses `atlassian-python-api` with REST API:
- No MCP size limits
- Handles attachments automatically
- Proper version management

See [troubleshooting_guide](troubleshooting_guide.md) for detailed error solutions.
