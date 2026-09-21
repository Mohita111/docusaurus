# Key Concepts: Static Site Generators and Content Types

This guide explains how this documentation platform turns written content into a published website. You do not need to know any programming to understand the core ideas.

## What is a static site generator?

A static site generator takes your written pages and builds a complete website of ready-to-serve files. It runs once at build time, producing a set of final HTML, CSS, and JavaScript files that can be hosted anywhere.

- You write content in simple text formats.
- The generator applies a chosen theme, navigation, and page layout.
- The result is fast and secure because there is no live database or server logic on every visit.

## Two generators in one project

This project includes two documentation approaches: Docusaurus and MkDocs Material. Both turn text into websites, but they suit different workflows.

| | Docusaurus | MkDocs Material |
|---|---|---|
| Primary strength | Product documentation, blog posts, and custom pages in one site | Straightforward documentation with a clean navigation |
| Content format | Markdown and MDX | Markdown |
| Navigation setup | Defined through a sidebar configuration | Defined through a navigation list in the site settings |
| Look and feel | Customizable with themes and extra pages | Material Design look with built-in search and copy-code features |

Docusaurus feels like a full website platform: documentation, blog posts, and fully custom pages can live together. MkDocs Material feels like a focused documentation site: you list the pages you want in the top navigation, and it builds a polished site around them.

## Content types in the Docusaurus area

Docusaurus organizes content into three main areas.

### 1. Documentation pages

These are the reference and guide pages. They live in a dedicated documents folder inside the site workspace. Each page can be a simple Markdown file or a more expressive MDX file, which allows Markdown to include interactive elements.

A documentation page becomes visible through the sidebar. The site includes a Tutorial section in the top navigation, and its sidebar is generated automatically from the documents folder.

### 2. Blog posts

Blog posts are dated articles. Each post can be a single file or a folder that groups the article with its images. The date is part of the post’s name, so the system knows when to show it.

For example, a post named with `2021-08-26` appears as an August 26, 2021 article. The blog can provide:

- A short list of authors
- Tags such as “welcome” or “tutorial”
- An optional summary before a “read more” break
- RSS and Atom feeds for readers who subscribe

### 3. Custom pages

Beyond documents and blog posts, the platform supports standalone custom pages. These are more flexible pages that are not tied to the documentation sidebar or the blog timeline. They can be used for landing pages, dashboards, or any one-off screen the site needs.

## How documentation sidebars work

A sidebar is the ordered list of links shown next to documentation pages. It gives readers a table of contents for a section and lets them move between pages.

In this project, the sidebar is set to generate automatically from the documents folder. This means:

- New documents appear in the sidebar without manually adding each one.
- The folder structure can influence the grouping and order.
- Readers get next and previous navigation from the sidebar.

Authors can also choose to define a sidebar manually if they want a specific order that differs from the folder layout.

## Front matter, tags, and authors

Each content page can begin with a small block of metadata called front matter. This is not the visible page content; it tells the platform how to treat the page.

Common metadata includes:

- **Title** — the page or post title shown in the browser and headings.
- **Slug** — a short, readable address for the page.
- **Authors** — the people credited with writing the post.
- **Tags** — keywords that group related posts.

In the blog example, the welcome post has:

- Authors: two named contributors
- Tags: three keywords including the platform name
- A summary marker that creates the preview text before the “read more” break

This metadata helps readers find related content, helps the site build author pages and tag archives, and keeps the displayed title separate from the file name.

## How MkDocs Material navigation works

The MkDocs Material area works from a navigation list in the site settings. This list maps human-friendly labels to the actual content files.

In this project, the main navigation has:

- A **Welcome** page as the home entry
- A **Core Integrations** section with four child pages:
  - Bitwarden Deployment
  - Storage with CSI for S3
  - Observability via OpenLIT
  - Database Cluster with Pangolin (CE)

Each label points to a specific Markdown file. The navigation list controls:

- The order of pages in the top menu
- Which pages appear under expandable sections
- The labels readers see, regardless of the raw file names

This is different from Docusaurus, where documentation navigation is a sidebar and blog posts have their own timeline.

## Source content vs. built output

It helps to think of the site in two states:

- **Source content** is what authors write and edit. These are the Markdown files, metadata blocks, images, and navigation settings. This is the working material, not what visitors download.

- **Built output** is what the static site generator produces. It is a folder of final pages, ready to be placed on a web host. This output is generated from the source content whenever the site is rebuilt.

The deployment process consumes the built output, not the source files. A typical flow is:

1. An author edits source content.
2. The static site generator runs and applies the theme, navigation, and metadata.
3. The generator writes the final website files.
4. The hosting service serves those final files to visitors.

Because visitors receive only the built output, the source content can include organizational folders, metadata, and drafts without exposing that structure to the published site.

## Where each type of content becomes a page

To anchor the concepts:

- **Docusaurus documentation** — each Markdown or MDX document becomes a page when the documentation sidebar includes it.
- **Docusaurus blog posts** — each dated blog file or folder becomes a blog page on the site’s blog timeline.
- **Docusaurus custom pages** — each custom page in the pages area becomes a standalone page.
- **MkDocs Material pages** — each Markdown file listed in the navigation becomes a page in the Material site’s navigation.

Understanding this flow, from source content through the generator to the built output, helps you see how the site is structured and where new content should be added.