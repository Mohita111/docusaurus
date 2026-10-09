# How to minify JavaScript bundles

Learn how to compress the JavaScript files in your Docusaurus site during the build process so visitors download smaller files and pages load faster.

## Prerequisites

- A Docusaurus project that builds successfully
- The `@docusaurus/faster` package added to your project dependencies
- Node.js and npm (or yarn) available in your terminal

## Enable faster JavaScript minification

### Step 1: Open your site configuration

Open the main configuration file at the root of your Docusaurus project.

### Step 2: Turn on the faster JavaScript compressor

In your site configuration, locate the `future` section. Add a `faster` entry that includes `swcJsMinimizer: true`:

```js
future: {
  faster: {
    swcJsMinimizer: true,
  },
},
```

This switches JavaScript compression to the SWC compressor, which runs faster than the default compressor while still producing well-compressed output.

### Step 3: (Optional) Adjust parallel compression

By default, JavaScript compression runs across multiple processor cores at the same time to speed up the build.

To control how many cores are used, set the `TERSER_PARALLEL` environment variable in your terminal before running the build:

- Run on 2 cores: `TERSER_PARALLEL=2 npm run build`
- Run on 4 cores: `TERSER_PARALLEL=4 npm run build`
- Turn off parallel compression: `TERSER_PARALLEL=false npm run build`
- Use all available cores (default): omit the variable entirely

### Step 4: Build your site

Run the build command in your terminal:

```
npm run build
```

Or if you use yarn:

```
yarn build
```

### Step 5: Check the results

After the build finishes, look in your site's output folder. The JavaScript files there should be compressed and ready to serve. You can compare file sizes or use your browser's developer tools to see how much data is being sent to visitors.

## Expected result

Your site's JavaScript files are automatically compressed during every build. Visitors download smaller files, which helps the site load faster.

## Common issues

### Build fails after you enable the faster JavaScript compressor

The faster JavaScript compressor requires the `@docusaurus/faster` package. If the package is not installed, the build displays an error explaining this requirement. To fix the error, add `@docusaurus/faster` to your project dependencies and rebuild.

### The build is slower than expected

Try explicitly setting the `TERSER_PARALLEL` environment variable to a positive number to force parallel compression. If your machine has limited CPU cores, parallel compression may already be working as well as it can.

### JavaScript output does not seem much smaller

JavaScript compression removes whitespace, shortens variable names, and applies other size reductions. If your JavaScript files are already small or contain few comments, the size reduction may be less noticeable. The compression is still working even when the reduction is modest.