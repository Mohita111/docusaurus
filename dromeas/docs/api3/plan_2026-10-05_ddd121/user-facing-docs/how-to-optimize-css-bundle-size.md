# How to optimize CSS bundle size

You can reduce the size of your CSS bundles during the build process by using Docusaurus's built-in CSS optimization features. This guide walks you through enabling and configuring CSS minification and cleanup.

## Prerequisites

- You have a Docusaurus project initialized and ready to build
- You have access to your project's configuration file (`docusaurus.config.js`)
- You're familiar with running build commands in your terminal

## Step-by-step instructions

### Step 2: Verify CSS minification is active

After the build completes, check that CSS files are minified:

1. Navigate to the`build` folder in your project
2. Look for`.css` files (usually in`build/_docusaurus/`)
3. Open any CSS file in a text editor and confirm the output is compressed (single-line, no whitespace)

CSS minification is enabled by default and removes comments, whitespace, and redundant declarations.

### Step 3: Remove unnecessary custom properties (optional)

If your site uses CSS custom properties (CSS variables) with overridden values, you can remove the redundant ones:

1. Open your`docusaurus.config.js` file
2. The CSS optimization already includes a custom property cleanup feature that removes duplicate variable declarations from the`:root` selector
3. This happens automatically during the build—no additional configuration needed

The optimizer intelligently keeps the final value of each custom property while removing earlier declarations that are overridden. If a property uses`!important`, it preserves important declarations and removes only the non-important duplicates.

### Step 4: Review the optimized bundle size

After the build completes, compare your CSS file sizes:

1. Locate the generated CSS files in your`build` folder
2. Check the file sizes compared to your previous builds
3. You can use browser developer tools to inspect which CSS files are loaded on your site

## Expected results

After completing these steps, you should see:

- CSS files compressed to a single line with no unnecessary whitespace
- Duplicate custom property declarations removed from the`:root` selector
- Overall CSS bundle size reduced by 20–40% depending on your original CSS complexity
- Faster page load times due to smaller CSS payloads
- Build output showing no CSS-related warnings

## Common issues

**CSS output doesn't look minified**
- Verify you ran`npm run build` (not`npm start`, which uses development mode)
- Check that you're looking at files in the`build` folder, not your source folder
- CSS optimization only applies to production builds

**Build time increased significantly**
- CSS optimization adds a small overhead to build time
- This is expected and pays off in runtime performance through smaller bundle sizes
- If optimization time is too long, ensure your CSS files don't have extreme complexity (thousands of rules)

**Custom properties aren't being removed**
- The optimizer only removes duplicate custom properties declared in the`:root` selector
- Properties in other selectors or media queries are left unchanged
- Verify your custom properties are defined in`:root` for the cleanup to apply

**CSS styling looks different after optimization**
- CSS minification should never change visual output
- If styling changed, check browser console for CSS errors
- Run`npm run build` again and verify the source CSS files are correct

## Related features

Optimize your overall bundle further by exploring these complementary features:

- CSS Optimization, Learn about all CSS optimization capabilities
- Performance Optimization, Enable additional build optimizations like minifying JavaScript
- Site Building & Bundling, Understand the complete build and bundling process
