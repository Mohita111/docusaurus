# Redirect broken or legacy URLs

If a page moved or changed its address, visitors using an old link can still land on the right page. Use client-side redirects to send broken or legacy addresses to a current page.

## Prerequisites

- You can edit the site configuration file (`docusaurus.config.js`) as a site administrator.
- The client redirects plugin is enabled in your site configuration.
- You know the old or broken addresses and the current page address each one should redirect to.

## Add a redirect

1. Open your`docusaurus.config.js` file.
2. In the`plugins` array, find the`@docusaurus/plugin-client-redirects` configuration.
3. Locate the`redirects` option within the plugin settings.
4. Add a new object to the`redirects` array with these properties:
   - Set`from` to the old or broken address (without the domain; for example,`/old-guide` or`old-guide`)
   - Set`to` to the address of an existing page (for example,`/docs/intro`)
5. To redirect multiple old addresses to the same page, use an array for`from` instead:`from: ['/v1-pricing', '/legacy-pricing']`
6. Repeat steps 4–5 for each old or broken address you want to redirect.
7. Save your configuration file.
8. Run`docusaurus build` to generate the redirect pages during the build process.
9. After the build completes, test by opening one of the old addresses in your browser to confirm it redirects to the destination page.

## Example configuration

```js
// docusaurus.config.js
module.exports = {
  plugins: [
    [
      '@docusaurus/plugin-client-redirects',
      {
        redirects: [
          {
            from: '/old-guide',
            to: '/docs/intro',
          },
          {
            from: ['/v1-pricing', '/legacy-pricing'],
            to: '/pricing',
          },
        ],
      },
    ],
  ],
};
```

## Expected result

When the build completes, each old address you listed is converted into a redirect page. Visitors who use an old link see a temporary redirect page that automatically forwards them to the destination page within seconds. Visitors do not need to update their bookmarks or links.

## Common issues

### The destination page does not exist

**Issue:** The build fails with an error about an invalid redirect target.

**Solution:** Verify that the`to` address matches an existing page on your site exactly, including any trailing slashes required by your configuration. Check your documentation structure and use the correct path.

### Trailing slash mismatch

**Issue:** The build fails because the`to` address does not match the trailing slash setting.

**Solution:** If your site has`trailingSlash: true`, all`to` addresses must end with a slash (for example,`/docs/intro/`). If your site has`trailingSlash: false`, addresses must not end with a slash (for example,`/docs/intro`). Update your redirect configuration to match this rule.

### The same old address appears in more than one redirect

**Issue:** You see a warning during the build that multiple redirects point from the same`from` address.

**Solution:** An old address can redirect to only one destination. Review all your`from` entries and remove duplicates. Keep only the redirect that points to the correct current page.

### The old address is already a live page

**Issue:** The build finishes but the redirect is not created, and you see a warning that the redirect would override an existing path.

**Solution:** The`from` address matches a page that actually exists on your site, so the redirect is ignored. Choose a different old address that is not currently in use.

### Redirects do not work on a hash-based site

**Issue:** The plugin is disabled and redirects are not created.

**Solution:** Client-side redirects do not work when your site uses the hash router (where URLs look like`example.com/#/page`). Use the default path-based routing instead, or consider alternative redirect solutions like server-side redirects on your hosting platform.

### Search query or anchor is lost during redirect

**Issue:** When a visitor clicks a link like`/old-page?search=term#section`, they arrive at the new page without the query string or anchor.

**Solution:** If your redirect's`to` address does not include its own query string or anchor, the redirect page automatically forwards any search terms and anchors from the old link. This happens transparently and visitors see the correct section or search context on the destination page.

## Related topics

Client-Side Redirects · Handle page renames without 404s
