# How to handle page renames without 404s

When you rename a page, links to the old address break and visitors see a 404 error. You can create a redirect to automatically send visitors from the old address to the new page instead.

## Prerequisites

- You have access to your site's configuration file (`docusaurus.config.js`)
- The new page already exists at its final address
- You know both the old page address and the new page address

## Set up a redirect for a renamed page

1. Open your site's`docusaurus.config.js` file.

2. Locate the`plugins` array and find the`client-redirects` plugin configuration. If it doesn't exist, add it as a new entry.

3. In the plugin configuration, add a`redirects` array if it doesn't already exist.

4. Add a new redirect object with the following structure:
```javascript
   {
     from: '/docs/old-page-name',
     to: '/docs/new-page-name'
   }
   ```
   Replace`/docs/old-page-name` with the old page address and`/docs/new-page-name` with the new page address.

5. If the renamed page was previously reachable at multiple old addresses, you can redirect all of them to the new address by using an array:
```javascript
   {
     from: ['/docs/old-page-name', '/docs/alternative-old-name'],
     to: '/docs/new-page-name'
   }
   ```

6. Save your changes to`docusaurus.config.js`.

7. Run the build command to generate your site with the new redirects:
```
   docusaurus build
   ```

8. Deploy or publish your site.

## Expected result

When a visitor opens the old page address in their browser, they're automatically redirected to the new page. The browser address bar updates to show the new address, and the visitor sees the page content without any error message.

## Common issues

### The redirect points to a page that doesn't exist

The redirect target must be a real page that exists on your site. If you redirect to a page address that hasn't been created, the build process will show an error and the redirect won't be generated. Verify that the address in your`to` field matches an actual page on your site.

### Multiple redirects have the same "from" address

You can't create two redirects with the same old address pointing to different new pages. If your configuration has duplicate`from` values, only one redirect will be created. Review your`redirects` array and remove any duplicate old addresses.

### Trailing slash mismatch causes redirect to fail

If your site is configured to require trailing slashes (when`trailingSlash: true` is set in your site config), make sure your`to` address includes the trailing slash:`/docs/new-page-name/`. If your site is configured to not use trailing slashes (when`trailingSlash: false`), write the`to` address without a trailing slash:`/docs/new-page-name`. The redirect plugin automatically validates this based on your site's trailing slash setting.

## Related

Client-Side Redirects, Learn more about how the redirect plugin works and other redirect strategies.
