# Quick Start Guide: Using SingleFile WebOps Workflows

This guide will help you get started with the GitHub Actions workflows for web operations.

## Step 1: Navigate to Actions

1. Go to your GitHub repository
2. Click on the **Actions** tab at the top
3. You'll see all available workflows in the left sidebar

## Step 2: Run Your First Capture

### Option A: Capture a Single Page

1. Click **Capture Single Web Page** in the left sidebar
2. Click the **Run workflow** dropdown on the right
3. Enter a URL (e.g., `https://example.com`)
4. (Optional) Enter a custom filename
5. Click the green **Run workflow** button
6. Wait for the workflow to complete (usually 30-60 seconds)
7. Click on the completed workflow run
8. Scroll down to **Artifacts**
9. Download **captured-page.zip**
10. Extract and view the HTML file

### Option B: Capture Multiple Pages

1. Click **Capture Multiple Pages** in the left sidebar
2. Click **Run workflow**
3. Enter URLs, one per line, in the text box:
   ```
   https://example.com
   https://example.org
   https://wikipedia.org
   ```
4. Click **Run workflow**
5. Download the **captured-pages** artifact when complete

### Option C: Crawl a Website

1. Click **Crawl and Archive Website**
2. Click **Run workflow**
3. Configure:
   - **URL**: Starting URL (e.g., `https://example.com`)
   - **Max depth**: How many levels deep to crawl (1-3 recommended)
   - **Inner links only**: Check to stay within the same domain
   - **Max parallel workers**: Number of concurrent captures (2-4 recommended)
4. Click **Run workflow**
5. Download **crawled-site** artifact when complete

## Step 3: Set Up Scheduled Archiving (Optional)

To automatically archive pages daily:

1. Go to `.github/workflows/scheduled-archive.yml`
2. Click the **Edit** button (pencil icon)
3. Find the `URLS=()` section around line 32
4. Replace example URLs with your own:
   ```bash
   URLS=(
     "https://your-important-site.com"
     "https://your-documentation.com"
     "https://your-blog.com/latest"
   )
   ```
5. Click **Commit changes**
6. The workflow will now run daily at 2 AM UTC automatically

## Step 4: Set Up Change Monitoring (Optional)

To monitor a page for changes:

1. Go to `.github/workflows/monitor-changes.yml`
2. Click the **Edit** button
3. Find line 54 where it sets the default URL
4. Replace `https://example.com` with your URL
5. Click **Commit changes**
6. The workflow will check for changes every 6 hours

To manually check a specific page:
1. Go to Actions → **Monitor Page Changes**
2. Click **Run workflow**
3. Enter the URL to monitor
4. The workflow will compare it with the previous capture

## Understanding Artifacts

**What are artifacts?**
- Artifacts are the captured HTML files produced by workflows
- They are stored in GitHub for 30-90 days (depending on the workflow)
- You can download them as ZIP files

**How to download:**
1. Go to Actions tab
2. Click on a completed workflow run
3. Scroll to the **Artifacts** section at the bottom
4. Click on the artifact name to download

**File locations inside the ZIP:**
- Single captures: `captured-pages/[filename].html`
- Multiple captures: `captured-pages/*.html`
- Crawled sites: `crawled-site/*.html` + `crawl-report.md`

## Creating Custom Workflows

Use the example template as a starting point:

1. Copy `.github/workflows/example-custom-workflow.yml`
2. Rename it (e.g., `my-custom-capture.yml`)
3. Edit the URLs and settings
4. Commit the file
5. Run it from the Actions tab

Example customization:
```yaml
- name: Capture my homepage
  uses: ./.github/actions/capture-page
  with:
    url: 'https://my-site.com'
    output-file: 'homepage.html'
    compress-html: 'true'
    remove-unused-styles: 'false'
```

## Common Options

### Composite Action Options

When using `.github/actions/capture-page`, you can configure:

- `url`: The webpage URL (required)
- `output-file`: Custom filename (optional)
- `browser-headless`: Run browser invisibly (default: true)
- `browser-wait-until`: When to consider page loaded
  - `networkIdle` (default): Wait for network to be idle
  - `load`: Wait for page load event
  - `domContentLoaded`: Wait for DOM ready
- `compress-html`: Reduce file size (default: true)
- `remove-unused-styles`: Remove unused CSS (default: true)
- `remove-unused-fonts`: Remove unused fonts (default: true)
- `additional-args`: Extra CLI arguments

### Browser Wait Times

For slow-loading pages, add delays:
```yaml
additional-args: '--browser-wait-delay=5000 --browser-load-max-time=120000'
```

### Mobile Emulation

Capture pages as they appear on mobile:
```yaml
additional-args: '--browser-mobile-emulation=true'
```

## Troubleshooting

### Workflow doesn't appear in Actions tab
- Make sure the YAML file is in `.github/workflows/`
- Check that it has `on: workflow_dispatch:` or a schedule
- Commit and push the file if you edited locally

### Page capture times out
- Increase timeout: `additional-args: '--browser-load-max-time=120000'`
- Try different wait condition: `browser-wait-until: 'load'`

### Missing images or styles
- Set `remove-unused-styles: 'false'`
- Increase `max-resource-size: '50'` (in MB)

### Large file sizes
- Enable `compress-html: 'true'`
- Set `remove-unused-styles: 'true'`
- Reduce `max-resource-size` to exclude large media

## Advanced Usage

### Capture authenticated pages

Add authentication headers:
```yaml
additional-args: '--http-header="Authorization: Bearer TOKEN"'
```

Or use cookies:
```yaml
additional-args: '--browser-cookies="session_id,abc123,example.com,/,,,true,Lax"'
```

### Capture with JavaScript enabled

By default, scripts are blocked. To enable:
```yaml
additional-args: '--block-scripts=false'
```

### Schedule at specific times

Edit the cron expression in scheduled workflows:
```yaml
schedule:
  - cron: '0 9 * * 1'  # Every Monday at 9 AM UTC
  - cron: '0 0 1 * *'  # First day of every month
```

## Next Steps

1. **Explore the Documentation**: Read `.github/workflows/README.md` for detailed information
2. **Review Examples**: Check the example workflow for patterns
3. **Customize**: Create your own workflows for specific needs
4. **Automate**: Set up schedules for regular archiving
5. **Monitor**: Use change detection for important pages

## Getting Help

- **Documentation**: See `.github/workflows/README.md`
- **Implementation Details**: See `WEBOPS_SUMMARY.md`
- **SingleFile CLI Options**: Run `./single-file --help`
- **GitHub Actions**: [GitHub Actions Documentation](https://docs.github.com/en/actions)

Happy archiving! 🎉
