# How to generate redirect pages at build time

You can automatically generate redirect pages during your build process to gracefully handle URL changes, restructured documentation, or legacy paths without manual intervention.

## Prerequisites

- Node.js 18 or later
- An existing Docusaurus project with the client redirects plugin enabled in your`docusaurus.config.js`
- Understanding of your site's URL structure and any paths that should redirect

## What happens during build

When you run the build command, the client redirects plugin analyzes all existing pages in your site and generates redirect HTML files based on your configuration. These files contain JavaScript that automatically forwards visitors from old URLs to new ones.

The plugin generates redirect pages by examining four sources:

1. **File extension rules**, automatically creates redirects between versions with and without extensions (for example,`/page``/page.html`)
2. **Custom redirects**, applies redirect rules you define explicitly
3. **Dynamic redirect generators**, runs custom functions you provide to create redirects for each page
4. **Path normalization**, applies your site's trailing slash settings consistently

## Steps to generate redirect pages

### 1. Configure redirect rules in your config file

Open`docusaurus.config.js` and locate or add the client redirects plugin configuration:

```javascript
module.exports = {
  // ... other config
  plugins: [
    [
      '@docusaurus/plugin-client-redirects',
      {
        redirects: [
          {
            to: '/docs/new-page-name',
            from: '/docs/old-page-name',
          },
          {
            to: '/blog/2024-update',
            from: ['/blog/old-post', '/blog/legacy-post'],
          },
        ],
      },
    ],
  ],
};
```

### 2. (Optional) add file extension rules

If your site uses special file extension handling, configure these to generate redirects automatically:

```javascript
[
  '@docusaurus/plugin-client-redirects',
  {
    fromExtensions: ['html', 'htm'],
    toExtensions: [''],
  },
]
```

### 3. (Optional) add dynamic redirect generator

For advanced scenarios, use a function to generate redirects programmatically. This function receives each page path and returns the old paths that should redirect to it:

```javascript
[
  '@docusaurus/plugin-client-redirects',
  {
    createRedirects(existingPath) {
      // For /docs/v1/api, also redirect from /api
      if (existingPath.includes('/v1/')) {
        return existingPath.replace('/v1/', '/');
      }
      return undefined;
    },
  },
]
```

### 4. Run the build command

Execute the production build:

```bash
npm run build
```

or if you use Yarn:

```bash
yarn build
```

### 5. Verify generated redirect files

After the build completes successfully, check the output directory (typically`build/`) for files created at redirect paths. Each redirect creates an HTML file that contains the forwarding logic.

For example, if you redirected from`/old-page` to`/new-page`, you'll find a redirect file at`build/old-page/index.html`.

## Expected result

When the build finishes:

- ✓ Redirect HTML files are created in your build output directory
- ✓ Console output confirms the number of redirects generated without errors
- ✓ Visitors accessing old URLs are automatically forwarded to new URLs with search queries and anchors preserved (when applicable)
- ✓ The site is ready to deploy with all redirects in place

## Common issues and solutions

**Redirects point to paths that don't exist**

The plugin validates that every redirect target (`to`) is an actual page that Docusaurus generates. If validation fails during build, check your redirect configuration for typos and ensure the target paths exist in your documentation structure.

**Multiple redirects with the same source path**

If you define more than one redirect from the same old path, the build logs a warning and uses only the first one. Restructure your redirects so each source path maps to exactly one target.

**Redirect target differs only in trailing slash**

If your site has`trailingSlash` set to`true` but your redirect points to`/page` instead of`/page/`, the build fails. Ensure your redirect targets match your site's trailing slash setting. You can enable trailing slash normalization in your plugin options, and the plugin will automatically adjust redirect paths to comply with your setting.

**Generated redirect files not being served**

Ensure your build output directory is deployed correctly. Redirect files must be accessible at the same URLs where visitors expect them. Some hosting platforms may need additional configuration to serve HTML files at directory paths.

**Search queries and anchors not forwarding**

By default, the plugin preserves search parameters and URL anchors when redirecting (for example,`?q=search` or`#section`). If your redirect target already includes search or anchor parameters, the original values are not forwarded. Use custom redirect functions if you need more control over this behavior.

## Related

Client-Side Redirects, Understand the full redirect system and available configuration options.

How to redirect broken or legacy URLs, Create explicit redirects for specific URL changes.

Build production site, Learn more about the build process and deployment.
