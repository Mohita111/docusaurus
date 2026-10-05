# Performance optimization

Performance Optimization enables developers to build faster Docusaurus sites by using advanced minification tools and targeting modern browsers. This feature reduces bundle sizes, speeds up build times, and improves overall site performance through optional build configurations.

## What it is

Performance Optimization is a suite of optional build enhancements that you can enable to make your Docusaurus site faster and smaller. It provides faster JavaScript and CSS minification using modern tools like SWC and Lightning CSS, along with the ability to target specific browser versions to reduce unnecessary polyfills and compatibility code.

## Who it's for

This feature is designed for developers who want to optimize their Docusaurus builds. Whether you're building a large documentation site, prioritize fast deployment, or want to reduce bandwidth usage, Performance Optimization gives you the tools to achieve those goals.

## What you can do

### Use faster JavaScript minification

Replace the default JavaScript minifier with SWC, a high-performance Rust-based minifier. SWC minification is significantly faster than traditional minifiers while producing equally small output. The minifier uses optimized settings including:

- ECMA 2020 target with ECMA 5 output for broad compatibility
- Mangle and compress options enabled for maximum size reduction
- ASCII-only output with no comments for consistent, portable builds
- Safari 10+ compatibility maintained

### Use faster CSS minification

Enable Lightning CSS as your CSS minifier to dramatically speed up CSS processing. Lightning CSS integrates with your site's browser targets to automatically apply only the CSS features and syntax your audience actually needs. For example, if you target modern browsers, it can remove older vendor prefixes and polyfills that modern browsers don't require.

### Target modern browsers

Configure your site to generate code only for browsers your audience actually uses. By narrowing your browser targets from "all browsers" to "modern browsers only," you reduce the amount of compatibility code included in your bundles. This means:

- Fewer polyfills for JavaScript features
- Simpler and smaller transpiled code
- Better performance on the browsers that matter most to your users

Your browser targets are automatically read from your project's`.browserslistrc` or`browserslist` field in`package.json`. You can also set custom targets per environment.

## Key capabilities

**Configurable browser targets**: Set different browser targets for client (browser) and server (Node.js) builds. The system automatically uses your project's browserslist configuration, or falls back to sensible defaults. For server builds, it targets your current Node.js version by default.

**Lazy minifier loading**: The HTML minifier is loaded only when needed during the build process, not during development. This keeps your development server fast and responsive.

**Automatic transpilation targeting**: The build system automatically detects and applies the right transpilation settings based on your browser targets. This happens transparently—you don't need to manually configure transpilation rules.

**Environment-specific optimization**: Server and client builds use different optimization settings. Client builds target browsers; server builds target Node.js. This ensures each build is optimized for its actual runtime environment.

## How it works

When you build your site with Performance Optimization enabled, the system:

1. Reads your browser target configuration (from browserslist or your configuration)
2. Uses that configuration to set minification targets for both JavaScript (via SWC) and CSS (via Lightning CSS)
3. Applies appropriate transpilation rules during the JavaScript compilation step
4. Generates minified output optimized for those specific targets

The process works the same way whether you're building for client browsers or a Node.js server, but with target-appropriate settings for each.

## Benefits

**Faster builds**: SWC minification is significantly faster than traditional JavaScript minifiers, reducing total build time especially for large sites.

**Smaller bundles**: By targeting modern browsers and applying aggressive minification, your generated bundles are noticeably smaller, leading to faster downloads and better site performance.

**Reduced bandwidth**: Smaller bundles mean less data transferred to your users, which reduces your hosting costs and improves user experience on slow connections.

**Automatic optimization**: Once configured, optimizations happen automatically during every build—no manual steps or tweaking required.

## Limits and considerations

**Browserslist configuration required**: Browser targeting relies on your project having a valid`.browserslistrc` file or`browserslist` field in`package.json`. Without this, Docusaurus uses its built-in defaults.

**Server target constraints**: For server builds, the system cannot target Node.js versions that browserslist doesn't know about yet. It automatically falls back to the most recent known Node.js version if your Node version is newer than browserslist's data.

**Build environment variables**: You can override server Node.js targeting with the`DOCUSAURUS_SERVER_NODE_TARGET` environment variable if needed for special deployment scenarios.

**Rspack bundler limitations**: When using Rspack bundler, browser target data is determined by Rspack's built-in browserslist integration, which may differ slightly from the full browserslist package.

## Related features

- Site Building & Bundling
- CSS Optimization
- Build Bundling and Optimization
