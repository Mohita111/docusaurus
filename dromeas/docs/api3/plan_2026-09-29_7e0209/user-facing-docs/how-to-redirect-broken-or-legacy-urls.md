# Redirect broken or legacy URLs

If a page moved or changed its address, visitors using an old link can still land on the right page. Use client-side redirects to send broken or legacy addresses to a current page.

## Prerequisites

- You can edit the site configuration as a site administrator.
- Client-side redirects are enabled for your site.
- You know the old or broken addresses and the current page address each one should open.

## Add a redirect

1. Open your site configuration.
2. Find the client redirects settings and locate the **Redirects** list.
3. Add a new redirect entry.
4. In **From**, enter the old or broken address that visitors still use.
5. In **To**, enter the address of an existing page where visitors should arrive.
6. To point several old addresses to the same page, add all of those addresses to the same **From** entry.
7. Repeat these steps for each old or broken address you want to redirect.
8. Save your configuration.
9. Rebuild your site so the redirect pages are created.
10. Open one of the old addresses in a browser to confirm it sends visitors to the destination page.

## Example

| Old address used by visitors | Current page they should reach |
|------------------------------|--------------------------------|
| `old-guide`                  | `docs/intro`                   |
| `v1-pricing` and `legacy-pricing` | `pricing`              |

## Expected result

Each old address you listed now opens a temporary redirect page and then immediately sends the visitor to the destination page. Visitors do not need to update their bookmarks or links.

## Common issues

### The destination page does not exist

The build fails if a **To** address is not an existing page. Check that you used the exact address of a current page, and rebuild.

### Trailing slash mismatch

If your site requires or removes trailing slashes from addresses, the **To** address must match that rule. For example, if your site always requires a trailing slash, use `docs/intro/` instead of `docs/intro`.

### The same old address appears in more than one redirect

A single old address can only point to one destination. Review the **From** entries and remove duplicate old addresses.

### The old address is already a live page

If a **From** address matches an existing page on your site, the redirect is ignored. Use an old or broken address that is not already in use.

### Redirects do not work on a hash-based site

Client-side redirects are disabled when your site uses hash-based navigation. Use a regular path-based site address for redirects.