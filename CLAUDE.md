# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a Claude Code skill for comprehensive Confluence documentation management. It provides Wiki Markup expertise, Markdown conversion, Mermaid diagram rendering, and integration with the Atlassian MCP server for direct Confluence API access.

## Core Architecture

### Skill Type: Project Skill
- **Activation**: Automatically loaded when handling Confluence-related tasks
- **MCP Integration**: Requires `mcp__atlassian` server configured with Confluence credentials
- **Primary Languages**: Python 3 for utility scripts, Markdown for documentation

### Key Components

1. **Format Conversion Engine** (`scripts/convert_markdown_to_wiki.py`)
   - Bidirectional Markdown ↔ Wiki Markup conversion
   - Handles headings, lists, tables, code blocks, links, images
   - Preserves inline formatting (bold, italic, code, strikethrough)
   - Edge case handling for nested structures

2. **Diagram Renderer** (`scripts/render_mermaid.py`)
   - Renders Mermaid diagrams to PNG/SVG using mermaid-cli
   - Extracts diagram blocks from Markdown files
   - Batch processing support with auto-numbering
   - Configurable themes, background colors, dimensions

3. **Metadata Generator** (`scripts/generate_mark_metadata.py`)
   - Adds mark CLI-compatible metadata headers to Markdown files
   - Supports space, title, parent, labels, attachments
   - Title inference from first H1 heading
   - Preserves or replaces existing metadata

4. **Reference Documentation** (`references/`)
   - Complete Wiki Markup syntax guide
   - Conversion rules and edge cases
   - mark CLI integration patterns
   - CQL (Confluence Query Language) examples

## Essential Commands

### Running Python Scripts

All scripts are standalone Python 3 utilities:

```bash
# Markdown to Wiki Markup conversion
python scripts/convert_markdown_to_wiki.py input.md [output.wiki]
python scripts/convert_markdown_to_wiki.py input.md  # Prints to stdout

# Mermaid diagram rendering
python scripts/render_mermaid.py diagram.mmd output.png
python scripts/render_mermaid.py -c "graph TD; A-->B" output.png
python scripts/render_mermaid.py --extract-from-markdown doc.md --output-dir diagrams/

# Generate mark metadata
python scripts/generate_mark_metadata.py file.md \
  --space DEV \
  --title "Page Title" \
  --parent "Parent Page" \
  --labels api,documentation
```

### Testing Scripts

No formal test suite exists. Manual testing workflow:

```bash
# Test conversion
python scripts/convert_markdown_to_wiki.py examples/sample-confluence-page.md

# Test Mermaid rendering (requires mermaid-cli installed)
echo "graph TD; A-->B" | python scripts/render_mermaid.py -c "graph TD; A-->B" test.png

# Test metadata generation
python scripts/generate_mark_metadata.py examples/sample-confluence-page.md --space TEST --title "Test"
```

## Integration Patterns

### MCP Tool Usage

When handling Confluence tasks, use MCP tools from `mcp__atlassian` server:

- **Search**: `confluence_search` with CQL queries
- **Read**: `confluence_get_page` by ID or title+space
- **Create**: `confluence_create_page` with space_key, title, content
- **Update**: `confluence_update_page` with page_id, version handling
- **Labels**: `confluence_add_label`, `confluence_get_labels`
- **Hierarchy**: `confluence_get_page_children` for page trees

### Conversion Workflow

Standard pattern for Markdown → Confluence:

1. Extract Mermaid diagrams from Markdown
2. Render diagrams to PNG/SVG using `scripts/render_mermaid.py`
3. Convert Markdown to Wiki Markup using `scripts/convert_markdown_to_wiki.py`
4. Upload diagram images as Confluence attachments
5. Replace Mermaid blocks with image references (`!diagram.png!`)
6. Create/update page via MCP with final Wiki Markup

### CQL Query Construction

Build CQL queries programmatically:

```python
# Space-scoped search
f'space = "{space_key}" AND type = page'

# Text search with date filtering
f'text ~ "{search_term}" AND created >= startOfYear()'

# Label-based with creator filter
f'label IN ("api", "docs") AND creator = currentUser()'

# Complex multi-criteria
f'space = "DEV" AND type = page AND label = "api" AND created >= now("-30d") ORDER BY created DESC'
```

## Format Conversion Reference

### Critical Conversion Rules

**Markdown → Wiki Markup:**
- `# Heading` → `h1. Heading` (ATX-style headings)
- `**bold**` → `*bold*` (bold syntax reversal)
- `*italic*` → `_italic_` (italic syntax swap)
- `` `code` `` → `{{code}}` (inline code)
- `[text](url)` → `[text|url]` (link syntax)
- `![alt](url)` → `!url|alt=alt!` or `!url!` (images)
- `- item` → `* item` (unordered lists)
- `1. item` → `# item` (ordered lists)
- Table headers: `| Header |` → `||Header||`
- Table cells: `| Cell |` → `|Cell|`

**Edge Cases to Handle:**
- Nested lists: Indentation level → repetition of list markers (`**` for 2nd level)
- Code blocks: Preserve language hints as `{code:language=python}`
- Task lists: `- [ ]` → `[]`, `- [x]` → `[x]`
- Blockquotes: `> text` → `bq. text`
- Horizontal rules: `---` → `----` (4 dashes minimum)

### Unsupported Conversions

- Markdown footnotes → No Wiki Markup equivalent
- GitHub-flavored task lists → Simplified checkbox syntax
- Confluence macros → Cannot reverse-convert to Markdown
- HTML embedded in Markdown → Passed through or stripped

## mark CLI Integration

### Configuration Location
`~/.config/mark` with TOML format:

```toml
username = "email@example.com"
password = "api-token"
base_url = "https://instance.atlassian.net/wiki"
space = "DEFAULT_SPACE"
```

### Metadata Header Format

Insert at top of Markdown files:

```markdown
<!-- Space: DEV -->
<!-- Parent: API Documentation -->
<!-- Title: Authentication Guide -->
<!-- Label: api -->
<!-- Label: authentication -->
<!-- Attachment: diagrams/auth-flow.png -->

# Authentication Guide
...content...
```

### Sync Commands

```bash
mark -f file.md                          # Sync with default config
mark -u user@email.com -p token -f file.md  # Explicit credentials
mark --dry-run -f file.md                # Preview changes
mark -c ~/.config/mark-prod -f file.md   # Custom config
```

## Optional Dependencies

### Required for Full Functionality

```bash
# Mermaid diagram rendering
npm install -g @mermaid-js/mermaid-cli

# mark CLI for Git → Confluence sync
brew install kovetskiy/mark/mark
# OR
go install github.com/kovetskiy/mark@latest

# Additional conversion tools (optional)
npm install -g markdown2confluence
```

### Verification

```bash
mmdc --version      # Check mermaid-cli
mark --version      # Check mark CLI
python3 --version   # Ensure Python 3.x
```

## Common Task Patterns

### Creating Confluence Page from Markdown

1. User provides Markdown content (possibly with Mermaid diagrams)
2. Call `extract_mermaid_from_markdown()` to find diagram blocks
3. Render each diagram: `render_mermaid(mermaid_code=code, output_path=path)`
4. Convert Markdown: `MarkdownToWikiConverter().convert(markdown_text)`
5. Replace Mermaid blocks with `!diagram-1.png!` references
6. Upload images as attachments via MCP
7. Create page: `confluence_create_page(space_key, title, content, content_format="wiki")`
8. Return page URL and ID

### Searching and Updating Pages

1. Build CQL query based on user criteria (space, title, labels, dates)
2. Execute: `confluence_search(query=cql_string, limit=N)`
3. Parse results, extract page_id
4. Fetch current: `confluence_get_page(page_id, include_metadata=True)`
5. Convert new content to Wiki Markup
6. Update: `confluence_update_page(page_id, title, content, version_comment)`

### Bulk Sync from Git Repository

1. Find all `.md` files in target directory
2. For each file:
   - Generate mark metadata with `generate_mark_metadata.py`
   - Extract and render Mermaid diagrams
   - Use mark CLI to sync: `mark -f file.md`
3. Maintain hierarchy using Parent metadata
4. Apply consistent labeling scheme

## Troubleshooting

### Script Execution Issues

- **Import errors**: Ensure running from repository root or use absolute paths
- **Permission errors**: Check file permissions with `ls -la scripts/`
- **Python version**: Scripts require Python 3.6+, use `python3` command

### Mermaid Rendering Failures

- Verify mermaid-cli installation: `mmdc --version`
- Test diagram syntax at https://mermaid.live
- Check output directory exists before rendering
- Try SVG format if PNG fails: `-f svg`

### MCP Connection Issues

- Confirm Atlassian MCP server is running and configured
- Verify Confluence credentials (API token, not password)
- Check base_url includes `/wiki` suffix
- Test with simple search before complex operations

### Conversion Artifacts

- Review output for unescaped special characters
- Check nested formatting (bold within italic, etc.)
- Verify table alignment with manual inspection
- Test code blocks with different language hints

## File Organization

```
.claude/skills/confluence/
├── SKILL.md                          # Complete skill documentation
├── README.md                         # Overview and quick start
├── QUICK_REFERENCE.md                # Cheat sheet
├── INSTALLATION.md                   # Installation guide
├── CLAUDE.md                         # This file
├── scripts/
│   ├── convert_markdown_to_wiki.py   # Markdown → Wiki Markup
│   ├── render_mermaid.py             # Mermaid → PNG/SVG
│   └── generate_mark_metadata.py     # mark metadata generator
├── references/
│   ├── wiki_markup_guide.md          # Wiki Markup syntax reference
│   ├── conversion_guide.md           # Conversion rules and edge cases
│   └── mark_tool_guide.md            # mark CLI documentation
├── examples/
│   └── sample-confluence-page.md     # Example Markdown file
└── assets/
    └── (diagram assets)
```

## Key Design Decisions

1. **Standalone Scripts**: All utilities work independently without skill framework dependencies
2. **Stateless Conversion**: No session state between conversions, pure functional approach
3. **MCP-First Integration**: Prefer MCP tools over direct API calls for reliability
4. **Metadata Separation**: mark metadata isolated in HTML comments, not embedded in content
5. **Graceful Degradation**: Scripts provide helpful error messages, don't fail silently

## Future Instance Guidelines

- Always validate CQL syntax before executing searches
- Confirm space access permissions before creating/updating pages
- Test conversions on small samples before bulk operations
- Use `--dry-run` with mark CLI before actual syncing
- Preserve user's existing metadata when updating files
- Render diagrams before conversion to avoid placeholder issues
- Include version comments when updating Confluence pages
