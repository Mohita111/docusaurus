# System requirements and prerequisites

Before you install Docusaurus, make sure your environment has the required software. This page explains what you need and how to check your setup.

## Node.js

Docusaurus requires **Node.js version 24.21 or later**.

To check your installed Node.js version, open a terminal and run:

```
node --version
```

If the reported version is lower than 24.21, upgrade Node.js before continuing.

## Package manager

You need a JavaScript package manager to create and manage a Docusaurus site. Docusaurus works with **pnpm**, **npm**, or **yarn**. The Docusaurus project itself is tested with **pnpm 12.3.4 or later**.

To check your package manager version, run one of these commands:

```
pnpm --version
```

```
npm --version
```

```
yarn --version
```

If you use pnpm, make sure the version is 12.3.4 or later.

## No database or backend required

Docusaurus is a static site generator. It builds plain HTML, CSS, and JavaScript files that you can host on any web server. You do not need to install a database, configure a backend server, or set up a server-side runtime.

## Optional dependencies

Some advanced features require additional accounts or services:

- **Search**: To add full-text search to your published site, you can sign up for an Algolia account. This is optional; your site works without it.

## Verify your environment

1. Open a terminal window.
2. Run `node --version`. Confirm the version is 24.21 or later.
3. Run `pnpm --version`, `npm --version`, or `yarn --version`. Confirm your preferred package manager is installed.
4. If both commands return version numbers, your environment is ready for installation.