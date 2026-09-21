# System Requirements and Prerequisites

Before you can build and preview the two documentation sites in this repository, your computer needs a few tools installed. The two sites use different technology stacks, so the requirements are listed separately below.

## Two Documentation Sites at a Glance

This repository contains two documentation sites:

- A Docusaurus website for general documentation.
- A DigitalOcean Solutions Docs site built with MkDocs and the Material theme.

Each site has its own set of requirements.

## Requirements for the Docusaurus Site

The Docusaurus site requires:

- Node.js version 20.0 or newer.
- A JavaScript package manager: Yarn or npm.
  - The project instructions use Yarn, but npm works with the same scripts.

### Check Node.js

Open a terminal and run:

```bash
node -v
```

If the version shown is `v20.0` or higher, you are ready. If it is lower, upgrade Node.js to version 20 or newer.

### Check your package manager

Check Yarn:

```bash
yarn --version
```

Or check npm:

```bash
npm --version
```

At least one of these should be installed.

When you run `yarn` or `npm install` inside the Docusaurus site, all other Docusaurus dependencies are installed automatically.

## Requirements for the DigitalOcean Solutions Docs Site

The DigitalOcean Solutions Docs site uses Python, MkDocs, and the Material for MkDocs theme.

You need:

- Python 3.12.
- pip, the Python package installer. pip normally comes with Python 3.12.
- MkDocs version 1.5.0 or newer.
- Material for MkDocs version 9.0.0 or newer.

### Check Python

Run:

```bash
python3 --version
```

You should see `Python 3.12.x`.

### Check pip

Run:

```bash
pip3 --version
```

A version number should appear.

### Install the required Python packages

If MkDocs and Material for MkDocs are not installed yet, install them with:

```bash
pip3 install "mkdocs>=1.5.0" "mkdocs-material>=9.0.0"
```

You can also use a Python virtual environment if you prefer to keep dependencies isolated.

## Git and GitHub Setup

To deploy the Docusaurus site to GitHub Pages, you need:

- Git installed on your computer.
- A GitHub account.
- Git configured with your GitHub username and email, especially if you deploy over HTTPS.
- If you prefer SSH deployment, an SSH key added to your GitHub account.

### Check Git

Run:

```bash
git --version
```

### Check your Git identity

Run:

```bash
git config user.name
git config user.email
```

If those are empty, set them with:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### Deployment authentication options

When you deploy the Docusaurus site, you can choose one of these options:

- Use SSH: set `USE_SSH=true` before running the deploy command.
- Use HTTPS: provide your GitHub username with `GIT_USER=your-username`.

This allows the deployment process to push the built site to the `gh-pages` branch.

## Recommended Editor Tooling

For writing and editing documentation content, use a modern text editor that supports Markdown and MDX.

Recommended setup:

- Use Visual Studio Code.
- Install a Markdown preview extension to see rendered content while you write.
- For Docusaurus content, use an MDX-aware extension for syntax highlighting.
- For the DigitalOcean Solutions Docs site, standard Markdown support is enough.

This makes it easier to author content for both sites and preview changes locally.

## Quick Readiness Checklist

| Tool or account | How to check | Ready when |
| --- | --- | --- |
| Node.js for Docusaurus | `node -v` | Version is 20.0 or newer |
| Yarn or npm | `yarn --version` or `npm --version` | A version number appears |
| Python for MkDocs | `python3 --version` | Version is 3.12.x |
| pip | `pip3 --version` | A version number appears |
| MkDocs package | `pip3 show mkdocs` | Version is 1.5.0 or newer |
| Material for MkDocs | `pip3 show mkdocs-material` | Version is 9.0.0 or newer |
| Git | `git --version` | A version number appears |
| GitHub account | Sign in at github.com | You can access your account |
| Text editor | Open your editor | Markdown or MDX support is available |

If all checks pass, your machine is ready to build and preview both documentation sites locally.