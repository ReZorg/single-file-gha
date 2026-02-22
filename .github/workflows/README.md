# SingleFile GitHub Actions Workflows

This repository includes a set of GitHub Actions workflows and a reusable composite action for web operations (webops) using the SingleFile CLI.

## Available Workflows

### 1. Capture Single Web Page (`capture-single-page.yml`)

**Purpose**: Manually capture a single web page on-demand.

**Trigger**: Manual (`workflow_dispatch`)

**Inputs**:
- `url` (required): URL to capture
- `output-file` (optional): Custom output filename
- `browser-headless` (optional, default: true): Run browser in headless mode
- `compress-html` (optional, default: true): Compress HTML output

**Usage**:
1. Go to Actions tab
2. Select "Capture Single Web Page"
3. Click "Run workflow"
4. Enter the URL and configure options
5. Download the captured page from artifacts

**Example**:
```
URL: https://example.com
Output file: example.html
Browser headless: true
Compress HTML: true
```

### 2. Capture Multiple Pages (`capture-multiple-pages.yml`)

**Purpose**: Capture multiple web pages from a list of URLs.

**Trigger**: Manual (`workflow_dispatch`)

**Inputs**:
- `urls` (required): URLs to capture (one per line)
- `browser-headless` (optional, default: true): Run in headless mode

**Usage**:
1. Go to Actions tab
2. Select "Capture Multiple Pages"
3. Click "Run workflow"
4. Enter URLs (one per line)
5. Download all captured pages from artifacts

**Example**:
```
URLs:
https://example.com
https://example.org
https://wikipedia.org
```

### 3. Scheduled Web Archive (`scheduled-archive.yml`)

**Purpose**: Automatically archive configured web pages on a schedule.

**Trigger**: 
- Schedule: Daily at 2 AM UTC
- Manual (`workflow_dispatch`)

**Configuration**:
Edit the workflow file to add URLs to archive:
```yaml
URLS=(
  "https://example.com"
  "https://your-site.com"
  "https://important-page.org"
)
```

**Artifacts**: Archives are stored for 90 days

### 4. Crawl and Archive Website (`crawl-website.yml`)

**Purpose**: Crawl a website starting from a URL and capture all linked pages.

**Trigger**: Manual (`workflow_dispatch`)

**Inputs**:
- `url` (required): Starting URL to crawl
- `max-depth` (optional, default: 1): Maximum crawl depth
- `inner-links-only` (optional, default: true): Crawl only internal links
- `max-parallel-workers` (optional, default: 4): Maximum parallel workers

**Usage**:
1. Go to Actions tab
2. Select "Crawl and Archive Website"
3. Click "Run workflow"
4. Configure crawl parameters
5. Download crawled site from artifacts

**Example**:
```
URL: https://example.com
Max depth: 2
Inner links only: true
Max parallel workers: 4
```

**Output**: Generates a crawl report with statistics and list of captured pages.

### 5. Monitor Page Changes (`monitor-changes.yml`)

**Purpose**: Monitor a web page for changes over time.

**Trigger**: 
- Schedule: Every 6 hours
- Manual (`workflow_dispatch`)

**Inputs** (manual trigger only):
- `url` (required): URL to monitor
- `check-interval` (optional, default: 24): Check interval in hours

**Features**:
- Compares current version with previous capture
- Detects changes automatically
- Stores timestamped versions when changes are detected
- Maintains latest version for comparison

**Configuration** (for scheduled monitoring):
Edit the workflow to set the default URL:
```yaml
# Default URL when triggered by schedule
echo "url=https://your-site.com" >> $GITHUB_OUTPUT
```

## Reusable Composite Action

### Capture Page Action (`.github/actions/capture-page`)

A reusable composite action that can be used in your own workflows.

**Usage**:
```yaml
- name: Capture web page
  uses: ./.github/actions/capture-page
  with:
    url: 'https://example.com'
    output-file: 'example.html'
    output-directory: './pages'
    browser-headless: 'true'
    compress-html: 'true'
```

**Inputs**:
- `url` (required): URL of the web page to capture
- `output-file` (optional): Output filename
- `output-directory` (optional, default: './captured-pages'): Output directory
- `browser-executable-path` (optional): Path to Chrome/Chromium
- `browser-headless` (optional, default: 'true'): Run in headless mode
- `browser-wait-until` (optional, default: 'networkIdle'): Page load condition
- `max-resource-size` (optional, default: '10'): Max resource size in MB
- `compress-html` (optional, default: 'true'): Compress HTML
- `remove-hidden-elements` (optional, default: 'true'): Remove hidden elements
- `remove-unused-styles` (optional, default: 'true'): Remove unused CSS
- `remove-unused-fonts` (optional, default: 'true'): Remove unused fonts
- `additional-args` (optional): Additional CLI arguments

**Outputs**:
- `output-path`: Path to the captured file

## WebOps Use Cases

These workflows enable various web operations scenarios:

### Content Archiving
- Archive important pages before they change or disappear
- Create periodic snapshots of web content
- Build a local archive of external documentation

### Change Monitoring
- Track changes to competitor websites
- Monitor regulatory or compliance pages
- Get notified when content updates

### Documentation Preservation
- Archive technical documentation
- Create offline copies of external resources
- Preserve reference materials

### Site Migration
- Capture entire websites for migration
- Create static backups of dynamic sites
- Archive legacy content

### Compliance & Legal
- Create timestamped evidence of web content
- Maintain historical records
- Document terms and conditions changes

## Advanced Configuration

### Custom Workflow Example

Create a custom workflow for your specific needs:

```yaml
name: Custom Archive Job

on:
  workflow_dispatch:

jobs:
  custom-archive:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: ./.github/actions/capture-page
        with:
          url: 'https://example.com'
          output-file: 'homepage.html'
          compress-html: 'true'
          
      - uses: ./.github/actions/capture-page
        with:
          url: 'https://example.com/about'
          output-file: 'about.html'
          
      - uses: actions/upload-artifact@v4
        with:
          name: site-archive
          path: captured-pages/
```

### Workflow Secrets

For advanced scenarios (e.g., authenticated pages), you can use GitHub secrets:

```yaml
- name: Capture authenticated page
  run: |
    ./single-file "${{ secrets.PROTECTED_URL }}" \
      --browser-cookie="${{ secrets.AUTH_COOKIE }}" \
      --output-directory=captured
```

## Requirements

All workflows automatically:
- Install Deno runtime
- Install Chromium browser (if not using custom path)
- Create necessary output directories
- Upload artifacts for download

## Artifacts

Captured pages are stored as GitHub Actions artifacts:
- Single page captures: 30 days retention
- Scheduled archives: 90 days retention
- Monitored page versions: 30-90 days retention

Download artifacts from the workflow run page in the Actions tab.

## Tips

1. **Large sites**: Use `max-parallel-workers` to control resource usage
2. **Slow pages**: Adjust `browser-wait-until` and wait delays
3. **Memory**: Increase `max-resource-size` for media-heavy pages
4. **Speed**: Disable `remove-unused-styles` and `remove-unused-fonts` for faster capture
5. **Size**: Enable `compress-html` to reduce file sizes

## Troubleshooting

### Browser not found
- The workflow automatically installs Chromium
- For custom browser, use `browser-executable-path` input

### Page timeout
- Increase timeouts with `additional-args: '--browser-load-max-time=120000'`
- Try different `browser-wait-until` values

### Large files
- Adjust `max-resource-size` to allow larger resources
- Use `compress-html` to reduce output size

## License

These workflows use SingleFile CLI which is licensed under AGPL. See LICENSE file for details.
