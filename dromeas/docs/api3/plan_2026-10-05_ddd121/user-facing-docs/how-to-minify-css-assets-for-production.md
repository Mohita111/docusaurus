# How to minify CSS assets for production

CSS minification reduces the file size of your stylesheets by removing unnecessary characters—spaces, comments, and redundant declarations—without changing how your styles work. This guide walks you through enabling CSS minification for your production build.

## Prerequisites

- A Docusaurus project set up and ready to build
- Basic familiarity with running build commands
- Access to your project's configuration file

## Why minify CSS

Smaller CSS files load faster in browsers, improving your site's performance and user experience. Docusaurus automatically minifies CSS during production builds using the CSSnano preset, which applies optimization techniques including:

- Removing duplicate and overridden custom properties
- Sorting media queries for better compression
- Eliminating unnecessary comments
- Removing redundant values

## Steps to minify CSS for production

1. **Open your terminal** and navigate to your Docusaurus project directory.

2. **Run the production build command**:
```bash
   npm run build
   ```

3. **Wait for the build to complete**. Docusaurus automatically applies CSS minification to all stylesheets during the build process.

4. **Check the output folder** in`build/` to verify your minified CSS files are present. The files will have`.css` extensions but with significantly reduced file sizes compared to development versions.

5. **(Optional) Customize minification behavior** by editing your`docusaurus.config.js` file if you need to adjust specific optimization settings:
```javascript
   module.exports = {
     // ... other config
     webpack: {
       jsLoader: (isServer) => ({
         // ... loader config
       }),
     },
   };
   ```

## Expected result

After running the build, your CSS files will be:

- Significantly smaller in file size
- Ready for production deployment
- Fully functional with all styles preserved

Your browser will download and parse smaller CSS files, resulting in faster page loads.

## Customizing CSS optimization

Docusaurus uses a specialized CSS minification preset that handles Docusaurus-specific optimizations:

- **Custom properties cleanup**: Duplicate custom properties (CSS variables) declared in`:root` are automatically removed, keeping only the final value that will actually be applied
- **Media query sorting**: Media queries are reorganized to improve compression efficiency
- **Comment removal**: All comments are stripped from the final output
- **Smart ident reduction**: Counter identifiers are preserved to prevent breaking custom line number features

These optimizations happen automatically during the build process—no additional configuration is required for basic minification.

## Common issues

**Issue**: CSS files are still large after building.

- **Cause**: You may have run the development server instead of the production build.
- **Solution**: Use`npm run build` (not`npm start`) to trigger minification.

**Issue**: Styling looks broken after minification.

- **Cause**: Very rare—minification is safe and preserves all functionality.
- **Solution**: Check your browser's developer tools for CSS errors. Clear your browser cache and rebuild.

**Issue**: Custom CSS properties aren't being removed as expected.

- **Cause**: Minification only removes overridden custom properties in the`:root` selector; others are preserved for safety.
- **Solution**: This is expected behavior and ensures your custom properties work correctly across your site.

## Related documentation

- CSS Optimization, Learn about CSS optimization features and benefits
- How to build production site, Complete guide to building your site for production
- How to optimize CSS bundle size, Strategies for reducing overall CSS bundle size
- Build Bundling and Optimization, Overview of how Docusaurus bundles and optimizes assets
