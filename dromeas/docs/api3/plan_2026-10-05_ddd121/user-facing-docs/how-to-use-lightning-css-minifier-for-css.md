# How to use lightning CSS minifier for CSS

Use Lightning CSS to minify your CSS assets during the production build. Lightning CSS is a fast, modern CSS minifier that reduces file size and optimizes your stylesheets for better performance.

## Prerequisites

- You have a Docusaurus project with a`docusaurus.config.js` file
- Your project includes CSS files that need minification
- You're building for production (not development)
- You have browserslist configuration in your project (or you're using defaults)

## Steps

1. **Open your project configuration file**

   Navigate to your`docusaurus.config.js` file in your project root.

2. **Locate the build optimization settings**

   Find the section in your configuration where build and bundler options are defined. This is typically where you configure the build process.

3. **Enable Lightning CSS minification**

   Add or update your build configuration to use Lightning CSS for CSS minification. The Lightning CSS minifier automatically reads your browserslist configuration to determine which browser features to preserve or optimize.

4. **Run a production build**

   Execute the production build command:
```
   npm run build
   ```
   or
```
   yarn build
   ```

5. **Verify minification in output**

   Check your build output directory (typically`build/`) to confirm that CSS files are minified. Look for significantly smaller file sizes compared to development builds.

## Expected result

Your CSS files are processed through Lightning CSS minification, which:

- Removes unnecessary whitespace, comments, and redundant rules
- Automatically targets the browsers specified in your browserslist configuration
- Reduces CSS bundle size for faster page loads
- Maintains CSS compatibility with your target browsers

For example, a CSS file of 50KB might reduce to 35KB after minification, depending on your stylesheet content.

## Common issues

**CSS file sizes don't change much**
- Your CSS might already be highly optimized or contain minimal whitespace. Lightning CSS focuses on semantic optimizations based on your target browsers.

**Build time is longer than expected**
- Lightning CSS performs additional analysis to optimize for your target browsers. This is normal and results in smaller, faster-loading stylesheets.

**Some CSS features are removed**
- Lightning CSS may remove vendor prefixes or features not needed for your target browsers. If you need to preserve specific features, adjust your browserslist configuration to include older browsers.

**Build fails with CSS parsing error**
- Ensure your CSS files are valid. Lightning CSS is strict about CSS syntax. Check for unclosed braces, invalid selectors, or other syntax issues in your stylesheets.

## Related information

- Performance Optimization, Learn about other build optimization strategies
- Site Building & Bundling, Understand how builds work end-to-end
- CSS Optimization, Explore other CSS optimization techniques
