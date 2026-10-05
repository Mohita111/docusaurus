# How to minify HTML output

Docusaurus automatically minifies the HTML pages it generates during a production build. This guide explains when minification happens, how to confirm it worked, and how to control it if needed.

## Minify HTML during a site build

HTML minification is enabled by default during production builds. You don't need to add any extra settings or packages—it happens automatically.

1. Open your terminal in the folder that contains your Docusaurus site.
2. Run the build command for your package manager:

```bash
   npm run build
   ```

   You can also use`yarn build` or`pnpm build`.

3. Let the build finish. During the build, Docusaurus processes every generated HTML page and removes unnecessary characters while keeping the page valid and interactive.
4. Check the generated output folder (typically`build/`). The HTML files inside are the minified versions of your pages.

## Expected result

After a successful build, your HTML files will be noticeably smaller than their unminified versions. The minified HTML generally includes:

- A shorter document type declaration (for example,`<!doctype html>`)
- No unnecessary`type` attributes on script and style tags
- Minified inline JavaScript, CSS, and JSON where present
- Quoted attribute values, which social network crawlers and some parsers require
- Head and body tags preserved, which social image crawlers and RDFa parsers depend on
- Comments that are required to keep interactive React pages working correctly during hydration

Docusaurus is intentionally conservative with minification to protect page behavior, search engine indexing, social previews, and client-side hydration. Some HTML may look less aggressively minified than other tools produce, but this protects your site's functionality.

## Common issues

**The build fails with an HTML minification error**

If HTML minification encounters unexpected content, the build stops and displays a message beginning with`HTML minification failed`. The error message indicates whether the issue is from the Terser or SWC minifier. Temporarily skip HTML minification by setting`SKIP_HTML_MINIFICATION=true` and rebuild to confirm whether minification is the cause of the failure.

**Social cards or metadata stop working after minification**

Docusaurus preserves quoted attributes, head and body tags, and certain HTML comments specifically to protect social cards and crawler parsing. If you apply additional HTML compression to the output (such as a reverse proxy or CDN minifier), verify that it preserves these elements. Re-run the build without extra compression to isolate whether the issue originates from Docusaurus minification.

**HTML still appears minified after setting the skip variable**

Clear your previous build output folder completely before rebuilding. Stale files from an earlier build may remain if the output directory was not cleaned. Run:

```bash
   rm -rf build && npm run build
   ```

**A crawler cannot read a generated page**

This is often caused by HTML compression applied outside Docusaurus after the build completes. Keep Docusaurus-generated HTML as-is to preserve the safeguards it includes for crawlers, social networks, and React hydration.

**Hydration errors appear in the browser console**

Docusaurus avoids removing HTML comments, empty attributes, and redundant attributes that React depends on during client-side hydration. If you see hydration mismatches after minification, skip minification to confirm the issue originates there. If it does, file an issue with details about your content structure, as this may indicate a case Docusaurus minification should handle more conservatively.

## Related topics

- Site Building & Bundling, Learn about the overall build and bundling process
- Performance Optimization, Explore other ways to optimize your build output
