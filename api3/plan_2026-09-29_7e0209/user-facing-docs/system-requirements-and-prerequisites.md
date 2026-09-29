# System Requirements and Prerequisites

Welcome! Before you install and run Docusaurus, take a few minutes to confirm that your computer meets the minimum software requirements. This page lists everything you need, shows how to verify your setup, and explains which traditional infrastructure you can skip entirely.

## Minimum software requirements

Docusaurus needs only two things to run:

- **Node.js** version 24.21 or later
- **A package manager** such as pnpm, npm, or Yarn

The recommended package manager for this release is **pnpm** version 12.3.4 or later.

| Requirement | Minimum version |
| --- | --- |
| Node.js | 24.21 or later |
| pnpm (recommended) | 12.3.4 or later |
| npm or Yarn | A recent version compatible with Node.js 24.21 or later |

If you already have an older version of Node.js installed, upgrade to 24.21 or later before continuing. Docusaurus will not start correctly on older Node.js versions.

## Verify your environment

Follow these steps to confirm that your computer is ready:

1. Open a terminal or command prompt.
2. Run `node --version`. You should see a version number that is **24.21 or later**.
3. Run `pnpm --version` if you plan to use pnpm. You should see **12.3.4 or later**.
4. If you prefer npm, run `npm --version`. You should see a recent npm version.
5. If you prefer Yarn, run `yarn --version`. You should see a recent Yarn version.

If any command fails or shows a version below the minimum, install or upgrade that tool before you proceed to installation.

## No database or backend required

Docusaurus is a static site generator. This means it builds your website into plain HTML, CSS, and JavaScript files that can be hosted almost anywhere.

You do **not** need:

- A database such as MySQL, PostgreSQL, or MongoDB
- A backend server such as Node.js, PHP, Ruby, or Python running in production
- A content management system

All of your content lives in files, and the final output is a set of static files that any web server or hosting service can serve.

## Optional dependencies for advanced features

The following items are not required for basic use, but they can enhance your site:

- **Algolia account** — If you want to add hosted search to your documentation, you need an Algolia account and the associated search credentials.
- **Translation services** — If you plan to localize your site into multiple languages, you may use a translation management tool such as Crowdin, but this is entirely optional.

You can create, build, and run a Docusaurus site without any of these optional services. Add them only when you are ready to use the corresponding feature.

## Next steps

Once your environment passes the version checks and you understand the optional services, you are ready to install Docusaurus and create your first site.