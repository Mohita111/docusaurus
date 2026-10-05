# How to build a production site

This guide explains how to generate an optimized, deployable version of your Docusaurus website.

## Prerequisites

- A working Docusaurus site that runs successfully on your computer
- Node.js installed and available from a terminal
- A package manager such as npm, Yarn, or pnpm

## Expected result

- The terminal reports that the build completed without errors.
- The`build` folder contains the final static HTML, CSS, JavaScript, and asset files for your website.
- Your site is ready to upload to a hosting provider or deployment service.

During the build process, your site automatically undergoes several optimization steps:

- **JavaScript minification**: Code is compressed to reduce file sizes and improve download speed
- **CSS minification**: Stylesheets are optimized and unnecessary declarations removed
- **HTML minification**: HTML output is compacted while preserving functionality
- **Asset optimization**: Images and other resources are processed for better performance

## Common issues

### Build fails with an error

If the build stops with an error, read the message carefully. It usually identifies the page, setting, or content that needs to be fixed. Correct the issue, then run the build command again.

### Warnings appear during the build

Warnings may appear in the terminal even when the build succeeds. Review each warning to check whether it affects your site, then address it if needed.

### Build takes a long time

Large sites with many pages and images can take longer to build. Keep the terminal open until the process finishes on its own.

## Related

- Site Building & Bundling
- Performance Optimization
- Logging & Diagnostics
