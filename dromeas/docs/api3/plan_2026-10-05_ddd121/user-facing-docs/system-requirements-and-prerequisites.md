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

You need a JavaScript package manager to create and manage a Docusaurus site. Docusaurus works with **pnpm**, **npm**, or **yarn**.

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

## Operating system support

Docusaurus runs on Linux, macOS, and Windows. No special setup is required for any of these platforms, use the same Node.js and package manager commands across all systems.

## Disk and memory requirements

Building a Docusaurus site requires minimal resources:

- **Disk space**: Allow at least 500 MB for a new project, including dependencies. Larger sites with extensive documentation may need more.
- **Memory**: Most builds complete with 512 MB of RAM. For sites with hundreds of pages, 1 GB of RAM is recommended.

## Git (optional but recommended)

While not strictly required, **git** is helpful for version control and deploying your site. If you plan to host on GitHub Pages or similar platforms, install git from git-scm.com.

## Optional dependencies for advanced features

Some features work better with additional tools or services:

- **Component overrides**: Use the`@docusaurus/swizzle` package to customize theme components. Install it when you need to modify Docusaurus components.
- **Search**: To add full-text search to your published site, you can set up an Algolia account. This is optional; your site works without it.
- **Faster builds**: Install the optional`@docusaurus/faster` package if you want to use Rspack bundler or SWC transpilation for faster build times.

## No database or backend required

Docusaurus is a static site generator. It builds plain HTML, CSS, and JavaScript files that you can host on any web server. You do not need to install a database, configure a backend server, or set up a server-side runtime.

## Verify your environment

1. Open a terminal window.
2. Run`node --version`. Confirm the version is 24.21 or later.
3. Run`pnpm --version`,`npm --version`, or`yarn --version`. Confirm your preferred package manager is installed.
4. If both commands return version numbers, your environment is ready for installation.

## Next steps

Once your environment is ready, install and set up your first Docusaurus project.
