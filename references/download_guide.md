# Download Guide

Complete guide for downloading Confluence pages to Markdown.

## Script Reference

**Script:** `scripts/download_confluence.py`

```bash
# Basic usage
python3 scripts/download_confluence.py PAGE_ID

# Options
--output-dir DIR        # Output directory (default: confluence_docs)
--download-children     # Include child pages in subdirectories
--save-html             # Save intermediate HTML for debugging
--page-ids-file FILE    # Load page IDs from file
--env-file PATH         # Custom credentials file
```

## Workflow Steps

### 1. Get Page ID

From Confluence URL:
```
https://company.atlassian.net/wiki/spaces/TEAM/pages/780369923/Page+Title
                                                        ^^^^^^^^^
                                                        Page ID
```

### 2. Configure Credentials

Create `.env` file:
```bash
CONFLUENCE_URL=https://yourcompany.atlassian.net
CONFLUENCE_USERNAME=your.email@company.com
CONFLUENCE_API_TOKEN=your_api_token
CONFLUENCE_OUTPUT_DIR=./confluence_docs
```

### 3. Execute Download

```bash
# Single page
python3 scripts/download_confluence.py 780369923

# With children
python3 scripts/download_confluence.py --download-children 780369923

# Multiple pages from file
python3 scripts/download_confluence.py --page-ids-file page_ids.txt
```

### 4. Review Output

```
confluence_docs/
├── Page_Title.md
├── Page_Title_attachments/
│   ├── image1.png
│   └── diagram.svg
├── Page_Title_Children/
│   ├── Child_Page.md
│   └── Child_Page_attachments/
└── download_results.json
```

## Output Format

Downloaded markdown includes YAML frontmatter:

```yaml
---
title: Page Title
confluence_url: https://company.atlassian.net/wiki/spaces/TEAM/pages/780369923
confluence:
  id: "780369923"
  space: TEAM
  version: 42
  labels: [api, documentation]
breadcrumb:
  - id: "123"
    title: Root
  - id: "456"
    title: Parent
parent:
  id: "456"
  title: Parent Page
  file: Parent_Page.md
children:
  - id: "789"
    title: Child Page
    file: Page_Title_Children/Child_Page.md
attachments:
  - id: "att1"
    title: diagram.png
    media_type: image/png
---
```

## Features

- **Confluence macro handling**: Code blocks preserve language hints
- **Children macro**: Converts to list of child pages
- **Image localization**: Attachment URLs → local file paths
- **Hierarchical download**: Child pages in subdirectories
- **Retry with backoff**: Handles transient API errors

## Debugging

Enable HTML debug mode:
```bash
python3 scripts/download_confluence.py --save-html PAGE_ID
```

Creates `_html_debug/` with:
- `original_*.html` - Raw API response
- `formatted_*.html` - Pretty-printed
- `transformed_*.html` - After macro conversion
- `original_*.md` - Before post-processing

See [troubleshooting_guide](troubleshooting_guide.md) for common issues.
