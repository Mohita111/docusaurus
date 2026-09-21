# Understanding the Repository Layout and Site Workflow

This repository contains two separate documentation websites plus a top-level guide. Each website focuses on a different type of content and uses a different publishing path. Understanding this split makes it easier to know where to edit something and how that change reaches readers.

## The Three Main Areas

| Area | What it contains | How it publishes |
| --- | --- | --- |
| Docusaurus site | Main project homepage, tutorial documentation, and blog | Manual deployment using a local command |
| DigitalOcean guides site | Step-by-step guides for deploying tools on DigitalOcean | Automatic deployment on every push to the main branch |
| Top-level user guide | A standalone introductory guide called Userguide | Not included in either automated website; read directly |

## The Docusaurus Site

The first site, built with Docusaurus, is the main website for the project. It includes:

- a homepage with the site title and tagline
- a documentation area reached through the **Tutorial** link in the top navigation
- a blog area reached through the **Blog** link
- footer links to the documentation introduction, blog, community resources, and GitHub

### Authoring and Previewing

Changes to the homepage, documentation pages, or blog posts are made within this site. To see changes before publishing, run the local development server from this site’s folder. The site opens in a browser, and most edits appear immediately without needing to restart anything.

### Building and Deploying

When you are ready to publish, use the deployment command from this site’s folder. The command can use either SSH or your GitHub username. It builds the final site and uploads it to the dedicated GitHub Pages publishing branch. This step is manual: it does not happen automatically when content is pushed to the main branch.

## The DigitalOcean Guides Site

The second site, built with MkDocs, hosts technical deployment and integration guides for DigitalOcean infrastructure. The landing page lists the available guides:

- **Bitwarden Deployment** — a self-hosted password vault backed by cloud block storage
- **CSI for S3** — mounting DigitalOcean Spaces storage inside Kubernetes using the Container Storage Interface driver
- **OpenLIT** — observability, performance tracking, and cost metrics for generative AI workloads
- **Pangolin (CE)** — database cluster architecture and scaling setup

### How Publishing Works

This site is fully automated through GitHub Actions. Whenever a change is pushed to the main branch, the automation:

1. checks out the latest content
2. installs the documentation tools, including MkDocs and the Material theme
3. builds the site
4. configures GitHub Pages
5. uploads the built site
6. publishes it to GitHub Pages

You do not need to run a local build or a manual deployment command for this site. Publishing happens automatically after changes are pushed.

## Where to Make a Change

Use this decision guide:

| You want to... | Edit this area |
| --- | --- |
| Update the project homepage, tutorial, product docs, or blog | The Docusaurus site |
| Add or modify a DigitalOcean deployment guide such as Bitwarden, CSI for S3, OpenLIT, or Pangolin CE | The DigitalOcean guides site |
| Update the top-level introductory guide | The root-level Userguide |

## End-to-End Publishing Paths

### Docusaurus Site Path

1. Edit the relevant content.
2. Run the local development server to preview changes.
3. When satisfied, run the deployment command.
4. The site is built and pushed to the GitHub Pages publishing branch.
5. The published site updates.

### DigitalOcean Guides Site Path

1. Edit the relevant guide page.
2. Push the change to the main branch.
3. GitHub Actions starts automatically.
4. The site is built and deployed to GitHub Pages.
5. The published site updates.

## Relationship Between the Homepage, Docs Sidebar, and Blog

Within the Docusaurus site, the homepage top navigation connects the main content areas:

- The **Tutorial** link opens the documentation area and displays the docs sidebar for navigating tutorial pages.
- The **Blog** link opens the blog section, which also appears in the footer.
- The footer repeats the main destinations, including the documentation introduction and blog.

The docs sidebar organizes the documentation, while the blog remains a separate section with its own link and feed options. The homepage acts as the central entry point to both.

## What About the Root User Guide?

A standalone guide called Userguide exists at the top level of the repository. It is not part of either automated website workflow and is intended to be read directly. Changes to it do not trigger either site’s deployment process.