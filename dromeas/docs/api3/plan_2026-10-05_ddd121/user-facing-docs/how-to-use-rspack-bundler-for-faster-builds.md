# How to use rspack bundler for faster builds

Use Rspack to speed up your Docusaurus site builds. Rspack is a faster bundler alternative to Webpack that can significantly reduce your build time, especially for large documentation sites.

## Prerequisites

- You have a Docusaurus project set up and running
- You're familiar with editing your site configuration file (`docusaurus.config.js`)
- The`@docusaurus/faster` package is installed as a dependency in your project

## Enable rspack bundler

1. Open your site's`docusaurus.config.js` configuration file

2. Locate or add the`future.faster` section in your config

3. Set`rspackBundler` to`true`:

```javascript
module.exports = {
  // ... other config
  future: {
    faster: {
      rspackBundler: true,
    },
  },
};
```

4. Save your configuration file

5. Run your build command:

```bash
docusaurus build
```

The build process now uses Rspack instead of Webpack, processing your site faster.

## Expected result

When you run`docusaurus build` or`docusaurus serve`, Docusaurus automatically uses Rspack as the bundler. You'll see build progress output in your terminal as the Rspack bundler compiles your site. The build should complete noticeably faster than with Webpack, particularly for larger sites.

The final output—your static HTML, JavaScript, and CSS files—remains the same. Rspack produces identical results to Webpack; it just does the work faster.

## Minification with rspack

When using Rspack, Docusaurus automatically applies optimized minification for your code:

- **JavaScript**: Uses SWC minifier (faster than Terser) by default
- **CSS**: Uses Lightning CSS minifier (faster than CssNano) by default

These minifiers are designed specifically for Rspack and provide both speed improvements during the build and smaller final bundle sizes.

## Optional: Configure JavaScript minification

By default, Rspack uses SWC for JavaScript minification. To customize this behavior:

1. In`docusaurus.config.js`, add the`swcJsMinimizer` option:

```javascript
module.exports = {
  future: {
    faster: {
      rspackBundler: true,
      swcJsMinimizer: true, // Enabled by default with Rspack
    },
  },
};
```

## Optional: Configure CSS minification

By default, Rspack uses Lightning CSS for CSS minification. To customize this behavior:

1. In`docusaurus.config.js`, add the`lightningCssMinimizer` option:

```javascript
module.exports = {
  future: {
    faster: {
      rspackBundler: true,
      lightningCssMinimizer: true, // Enabled by default with Rspack
    },
  },
};
```

## Optional: Use SWC for JavaScript transpilation

For even faster builds, you can also enable SWC as your JavaScript loader:

1. In`docusaurus.config.js`, add the`swcJsLoader` option:

```javascript
module.exports = {
  future: {
    faster: {
      rspackBundler: true,
      swcJsLoader: true,
    },
  },
};
```

This replaces Babel with SWC for transpiling your JavaScript files, further accelerating the build process.

## Common issues

**Issue: "To enable Docusaurus Faster options, your site must add the @docusaurus/faster package as a dependency"**

**Solution**: You need to install the`@docusaurus/faster` package. Run:

```bash
npm install @docusaurus/faster
```

or if you use Yarn:

```bash
yarn add @docusaurus/faster
```

---

**Issue: Build fails after enabling Rspack**

**Solution**: Ensure your`docusaurus.config.js` syntax is correct. The`future.faster.rspackBundler` option must be a boolean. Clear your build cache and try again:

```bash
rm -rf build/ node_modules/.cache/
docusaurus build
```

---

**Issue: Rspack doesn't seem faster**

**Solution**: Build performance gains are most noticeable on larger sites with many pages. For small documentation sites, the difference may be minimal. Rspack shines when you have hundreds of pages or complex content structures.

## Related topics

- Build production site
- Performance Optimization
- Site Building & Bundling
