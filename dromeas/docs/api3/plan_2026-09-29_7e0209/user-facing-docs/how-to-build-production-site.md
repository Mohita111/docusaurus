# How to build production site

This guide explains how to generate an optimized, deployable version of your Docusaurus website.

## Prerequisites

- A working Docusaurus site that runs successfully on your computer
- Node.js installed and available from a terminal
- A package manager such as npm, Yarn, or pnpm

## Steps

1. Open your website project folder in a terminal.
2. Run the production build command: `npm run build`. If you use Yarn, run `yarn build`. If you use pnpm, run `pnpm build`.
3. Wait for the build to finish. The terminal shows progress while the site is being compiled and optimized.
4. When the build completes successfully, find the generated output in the `build` folder inside your project.
5. Optional: preview the production output locally by serving the `build` folder with the command `npm run serve`.

## Expected result

- The terminal reports that the build completed without errors.
- The `build` folder contains the final static HTML, CSS, JavaScript, and asset files for your website.
- Your site is ready to upload to a hosting provider or deployment service.

## Common issues

### Build fails with an error

If the build stops with an error, read the message carefully. It usually identifies the page, setting, or content that needs to be fixed. Correct the issue, then run the build command again.

### Error message mentions `sass`

If the terminal reports that it cannot find the `sass` module, install Sass in your project using `npm install sass`. If you use Yarn, run `yarn add sass`.

### Warnings appear during the build

Warnings may appear in the terminal even when the build succeeds. Review each warning to check whether it affects your site, then address it if needed.

### Build takes a long time

Large sites with many pages and images can take longer to build. Keep the terminal open until the process finishes on its own.