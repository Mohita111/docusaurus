# How to use SWC minifier for JavaScript

Use the SWC minifier to compress your JavaScript bundles during the production build process. This approach reduces bundle size and improves site performance while maintaining code functionality.

## Prerequisites

- Your Docusaurus project is set up and running
- You're using a recent version of Docusaurus with SWC support
- You have access to your`docusaurus.config.js` configuration file
- Node.js is installed on your system

## Steps

1. **Open your Docusaurus configuration file**

   Navigate to and open the`docusaurus.config.js` file in your project root directory.

2. **Locate the build configuration section**

   Find the`swcMinifyOptions` or`minify` configuration object within your config file. If this section doesn't exist, you'll add it in the next step.

3. **Enable SWC minification with standard settings**

   Add or update the minification configuration to use SWC. The default SWC minifier settings include:
   - ECMAScript 2020 compatibility level
   - Compression enabled with ECMAScript 5 target
   - Full module minification
   - Automatic name mangling to reduce variable name sizes
   - Safari 10 compatibility preservation
   - Unicode characters converted to ASCII-only output
   - Comments removed from output

4. **Save your configuration changes**

   Save the`docusaurus.config.js` file.

5. **Run the production build command**

   Open your terminal and run the build command to generate your optimized production site:

```
   npm run build
   ```

   or

```
   yarn build
   ```

6. **Verify the minified output**

   After the build completes, check the generated JavaScript files in the`build` directory. The files should be significantly smaller than the unminified versions.

## Expected result

When the build finishes successfully, you'll see:
- Reduced JavaScript file sizes in your production bundle
- A`build` directory containing optimized JavaScript files
- Build completion messages in your terminal confirming the process finished
- JavaScript bundles that maintain full functionality while being smaller

## Common issues

**Issue: Build fails with SWC errors**
- Ensure your JavaScript code uses syntax that SWC supports (ECMAScript 2020 or earlier)
- Check that all TypeScript in your project is valid

**Issue: Minified code doesn't work correctly**
- SWC preserves functionality by default; if errors occur after minification, verify your source code has no syntax errors
- Clear your build cache by deleting the`.docusaurus` folder and rebuild

**Issue: Build takes longer than expected**
- SWC minification is optimized for speed; longer build times may indicate other processing is occurring
- Check your system resources and available memory

## Related resources

Performance Optimization, Learn about other build optimization techniques available in Docusaurus.

How to build production site, Understand the complete production build workflow.

How to minify JavaScript bundles, Explore JavaScript minification in detail.

Site Building & Bundling, Discover the complete bundling system that uses SWC minification.
