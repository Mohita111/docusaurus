# How to configure redirect rules for localized paths

Configure client-side redirects to handle localized documentation paths and ensure visitors find content regardless of language-specific URL changes.

## Prerequisites

Before you begin, you'll need:

- Access to your Docusaurus configuration file (`docusaurus.config.js`)
- The Client-Side Redirects plugin installed and enabled in your project
- Knowledge of your site's existing page paths and localized URL structure
- Understanding of which old paths should redirect to new localized paths

## Configure redirects for localized paths

Follow these steps to set up redirect rules that handle localized documentation paths:

1. Open your`docusaurus.config.js` file in your project root

2. Locate the plugins array in your configuration

3. Find or add the`@docusaurus/plugin-client-redirects` plugin entry

4. In the plugin's options, add a`redirects` array if it doesn't exist:

```javascript
plugins: [
  [
    '@docusaurus/plugin-client-redirects',
    {
      redirects: [
        // Add your redirect rules here
      ],
    },
  ],
],
```

5. Add individual redirect rules for each localized path change. For each rule, specify the old path in the`from` field and the new localized path in the`to` field:

```javascript
redirects: [
  {
    from: '/docs/introduction',
    to: '/docs/fr/introduction',
  },
  {
    from: '/docs/getting-started',
    to: '/docs/es/getting-started',
  },
],
```

6. To redirect multiple old paths to a single new localized path, use an array in the`from` field:

```javascript
redirects: [
  {
    from: ['/docs/intro', '/docs/getting-started', '/docs/start-here'],
    to: '/docs/de/intro',
  },
],
```

7. Save your configuration file

8. Run`docusaurus build` to generate the redirect pages at build time

9. Deploy your site; redirect pages are now created in your build output

## Expected result

After completing these steps, visitors who access any old path listed in your`from` field will automatically be redirected to the corresponding new localized path in the`to` field. Redirect pages are generated as static HTML files during the build process and placed in your output directory.

When a visitor lands on an old URL like`/docs/introduction`, they're immediately redirected to the new localized path`/docs/fr/introduction`. If the visitor had added search parameters or anchor links to their URL, these are preserved and forwarded to the new path automatically.

## Common issues and solutions

**Redirect to non-existent paths**

If you see an error stating that your redirect target doesn't exist, verify that the path in your`to` field exactly matches an existing page in your site. Check the path spelling, trailing slashes, and language prefix.

**Trailing slash mismatches**

If your`to` path has different trailing slash behavior than your site configuration expects, adjust the path to match your site's`trailingSlash` setting. For example, if`trailingSlash` is set to`true`, ensure your`to` paths end with a slash:`/docs/fr/introduction/` instead of`/docs/fr/introduction`.

**Duplicate redirect rules**

If multiple rules have the same`from` path, only the first one is used. Remove duplicate entries and consolidate multiple old paths into a single rule using an array format in the`from` field.

**Redirects overriding existing pages**

If a`from` path already exists as an actual page in your site, the redirect is ignored to prevent conflicts. Ensure your`from` paths don't match any real documentation pages.

**Search and anchor forwarding not working**

Search parameters and anchor fragments are automatically forwarded only if your`to` path doesn't already include them. If you include`?search=term` or`#section` in your`to` path, user-provided search and anchors won't be appended.

---

## Related topics

- Client-Side Redirects, Overview of the redirect system and its capabilities
- How to redirect broken or legacy URLs, Set up redirects for deprecated page paths
- How to handle page renames without 404s, Manage URL changes when restructuring content
- How to localize documentation paths, Configure localized documentation structure
