# How to minify JavaScript bundles

Learn how to compress the JavaScript files in your Docusaurus site during the build process so visitors download smaller files and pages load faster.

## Prerequisites

- A Docusaurus project that builds successfully
- Node.js and npm (or yarn) available in your terminal
- The`@docusaurus/faster` package added to your project dependencies (required only for SWC minification)

## Enable faster JavaScript minification with SWC

### Step 1: Open your site configuration

Navigate to`docusaurus.config.js` in the root of your Docusaurus project and open it in your editor.

### Step 2: Enable SWC JavaScript minification

In your configuration file, locate or create the`future` section. Add the`faster` object with`swcJsMinimizer: true`:

```js
module.exports = {
  // ... other config
  future: {
    faster: {
      swcJsMinimizer: true,
    },
  },
};
```

The SWC minifier compresses JavaScript faster than the default Terser minifier while producing equally well-optimized output.

### Step 3: Build your site

Run the build command:

```bash
npm run build
```

Or if you use yarn:

```bash
yarn build
```

The build now uses SWC to minify all JavaScript bundles.

### Set the number of parallel workers

Run the build with a specific number of cores:

```bash
TERSER_PARALLEL=2 npm run build
```

Replace`2` with the number of CPU cores you want to use.

### Disable parallel compression

To run compression on a single core, set the variable to`false`:

```bash
TERSER_PARALLEL=false npm run build
```

### Use all available cores (default)

If you don't set`TERSER_PARALLEL`, the minifier automatically uses all available CPU cores.

## Expected result

After the build completes successfully, your site's`build` folder contains minified JavaScript files. File sizes are noticeably smaller than the source files. When visitors load your site, they download these compressed bundles, which improves page load performance.

You can verify minification by opening your browser's developer tools, navigating to the Network tab, and checking the size of JavaScript files downloaded.

## Common issues

### Build fails with "must add the @docusaurus/faster package"

The SWC minifier requires the`@docusaurus/faster` package as a dependency. Install it by running:

```bash
npm install --save-dev @docusaurus/faster
```

Then rebuild your site.

### JavaScript files are not noticeably smaller

Minification removes whitespace, shortens variable names, and optimizes code structure. The size reduction depends on your source code. Files that already contain minimal comments or are very small may show less dramatic size changes. Minification is still active; the compression is working even if the reduction is modest.

### Build is slower with parallel compression enabled

Parallel compression uses multiple CPU cores, which adds overhead for some machines. If you have limited CPU cores or the build is slower, try disabling parallel compression:

```bash
TERSER_PARALLEL=false npm run build
```

Alternatively, explicitly set`TERSER_PARALLEL` to a lower number, such as`2`, to reduce overhead.

## Related documentation

- Site Building & Bundling
- Performance Optimization
- How to build production site
- How to use SWC minifier for JS
