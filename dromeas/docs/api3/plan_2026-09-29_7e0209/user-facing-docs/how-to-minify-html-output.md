# How to minify HTML output

Docusaurus automatically minifies the HTML pages it generates during a production build. This guide explains when minification happens, how to confirm it worked, and how to control it if needed.

## Prerequisites

- An existing Docusaurus site
- A terminal where you can run build commands
- Permission to set environment variables if you want to disable minification

## Minify HTML during a site build

HTML minification is enabled by default. You do not need to add any extra settings or packages.

1. Open your terminal in the folder that contains your Docusaurus site.
2. Run the build command for your package manager, for example:

   ```bash
   npm run build
   ```

   You can also use `yarn build` or `pnpm build`.

3. Let the build finish. During the build, Docusaurus processes every generated HTML page and removes unnecessary characters while keeping the page valid and interactive.
4. Open the generated output folder. The HTML files inside are the minified versions of your pages.

## Expected result

After a successful build, the generated HTML files will be smaller than they would be without minification. The HTML will generally include:

- A shorter document type declaration
- No `type` attribute on script and style tags that browsers already understand
- Minified inline JavaScript, CSS, and JSON where present
- Quoted attribute values, which some parsers and social network crawlers require
- Head and body tags, which some crawlers and social image features depend on
- Comments that are needed to keep interactive pages working correctly

Some parts of the HTML may look less aggressive than other minifiers, because Docusaurus intentionally preserves content that can break page behavior, search engines, social previews, or hydration if removed.

## Skip HTML minification

You may want to skip HTML minification if you are troubleshooting a build issue or need to compare output sizes.

1. Open your terminal in the site folder.
2. Set the environment variable `SKIP_HTML_MINIFICATION` to `true`.
3. Run the build command while the variable is active.

   For example:

   ```bash
   SKIP_HTML_MINIFICATION=true npm run build
   ```

4. Build as usual. The generated HTML files will be emitted without HTML minification.

## Common issues

**The build fails with an HTML minification error**

If HTML minification encounters unexpected content, the build stops and displays a message that begins with `HTML minification failed`. Temporarily skip HTML minification and rebuild to confirm whether minification is the cause.

**Social cards or metadata stop working after minification**

Docusaurus keeps quoted attributes, head and body tags, and certain comments to protect social cards and crawler parsing. If you use an external HTML minifier on the output, verify that it preserves these items.

**HTML still appears minified after setting the skip variable**

Clear your previous build output and run the build again. Some stale files may remain from an earlier build if the output folder was not cleaned.

**A crawler cannot read a generated page**

This is often caused by additional HTML compression outside Docusaurus. Keep Docusaurus-generated HTML as-is to preserve the safeguards it includes.