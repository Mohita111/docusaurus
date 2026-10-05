# How to enable faster build mode

Enable faster build mode to significantly speed up your Docusaurus build process using advanced bundling and minification techniques.

## Goal

You want to build your documentation site faster by using optimized tooling for bundling, transpilation, and code minification.

## Prerequisites

- You have a Docusaurus project set up and can access your`docusaurus.config.js` file
- You're familiar with opening and editing configuration files
- Your Node.js version supports the faster build tools (Node.js 22.0.0 or later recommended)

## What faster build mode does

Faster build mode uses high-performance tools including:

- **Rspack** bundler: A faster alternative to Webpack for bundling your site
- **SWC** transpiler and minifier: Rapidly transpiles TypeScript and JSX, and minifies JavaScript output
- **Lightning CSS** minifier: Optimizes your CSS files for production

These tools work together to reduce build time while maintaining output quality.

## Enable faster build mode

1. Open your **docusaurus.config.js** file in your project root

2. Locate the configuration object (the main export)

3. Add or update the`bundler` property to use **'rspack'**:

```javascript
   module.exports = {
     // ... other config
     bundler: 'rspack',
     // ... rest of config
   };
   ```

4. (Optional) Configure SWC minifier options for JavaScript by adding a`minify` property:

```javascript
   module.exports = {
     bundler: 'rspack',
     minify: 'swc',
     minifyOptions: {
       jsMinifyOptions: {
         ecma: 2020,
         compress: true,
         mangle: true,
       },
     },
     // ... rest of config
   };
   ```

5. (Optional) Configure CSS minification using Lightning CSS by adding a`cssMinifyOptions` property:

```javascript
   module.exports = {
     bundler: 'rspack',
     cssMinifyOptions: {
       // Lightning CSS will target browsers based on your browserslist config
     },
     // ... rest of config
   };
   ```

6. Save your **docusaurus.config.js** file

7. Run your build to use the faster build mode:

```bash
   npm run build
   ```

   Or for development with faster rebuilds:

```bash
   npm start
   ```

## Expected result

Your build should complete noticeably faster than before. You'll see improvements especially when:

- Building large documentation sites with many pages
- Running repeated builds during development
- Minifying JavaScript and CSS for production

The final output quality remains the same—only the build speed improves.

## Common issues and solutions

**Build times haven't improved significantly**
- Ensure you're using Rspack as your bundler (check`bundler: 'rspack'` in your config)
- Verify you're on Node.js 22.0.0 or later for optimal performance
- Clear your build cache by deleting the`.docusaurus` folder and rebuilding

**SWC minifier produces different output than expected**
- SWC uses modern JavaScript targets by default (ES2020)
- If you need to support older browsers, adjust the`ecma` option in`minifyOptions`
- Check that your`browserslist` configuration matches your actual target browsers

**CSS not minifying with Lightning CSS**
- Verify your **browserslist** configuration exists in your project (usually in`package.json` or a`.browserslistrc` file)
- Lightning CSS uses this to determine which CSS features it can safely remove

---

## Related

- How to use Rspack bundler for faster builds
- How to use SWC minifier for JS
- How to use Lightning CSS minifier for CSS
- Performance Optimization
