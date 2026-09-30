# How to handle page renames without 404s

When you rename a page, links that use the old address can stop working. This guide shows you how to create a redirect so visitors who open the old address are sent to the new address instead of seeing a 404 page.

## Prerequisites

- You can edit your site’s redirect settings.
- You know the old page address and the new page address.
- The new page already exists.

## Add a redirect for a renamed page

1. Open your site’s configuration settings.
2. Find the **Client Redirects** section.
3. Add a new entry under **Redirects** with these values:
   - **From**: the old page address, for example `/docs/old-page-name`
   - **To**: the new page address, for example `/docs/new-page-name`
4. If the page was previously reachable at more than one address, add each old address as a separate **From** value under the same **To** address.
5. Save your changes.
6. Rebuild and publish your site.
7. Open the old page address in a browser.

## Expected result

Visitors who use the old link will arrive at the new page automatically. The old address will no longer display a “not found” message.

## Common issues

### The redirect target page is missing

Make sure the **To** address points to a page that already exists. Redirects to missing pages are not created.

### Two redirects use the same old address

If two rules have the same **From** address, only one redirect will be kept. Review your redirect list and remove duplicate **From** values.

### Trailing slash mismatch

If your site is configured to always use trailing slashes, write the **To** address with a trailing slash, such as `/docs/new-page-name/`. If your site is configured to not use trailing slashes, write the **To** address without one.