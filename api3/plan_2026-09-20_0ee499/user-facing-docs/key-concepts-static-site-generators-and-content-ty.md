# Key Concepts: Static Site Generators and Content Types

This page explains how content becomes a published website on this platform. You do not need to know any programming to understand the basic mental model, but it helps to know the difference between the content you write and the website your readers actually visit.

---

## What Is a Static Site Generator?

A static site generator turns written content and configuration into a finished website before anyone visits it. The final website consists of plain files — pages, styles, and scripts — that can be hosted almost anywhere. There is no database or server-side processing needed when a reader opens a page.

This approach has several benefits:

- Pages load quickly because they are already prepared.
- The site is more secure because there is no dynamic code running behind the scenes.
- Content can be reviewed and versioned like any other set of files.
- Publishing is simply a matter of rebuilding the site and copying the output to a hosting service.

Two static site generators are relevant to this platform: **Docusaurus** and **MkDocs Material**. They produce similar results but are used for different kinds of sites.

---

## Docusaurus vs. MkDocs Material

| | Docusaurus | MkDocs Material |
|---|---|---|
| Best for | Product documentation, blogs, and community sites | Solution guides and structured technical documentation |
| Navigation style | Automatic sidebars generated from content folders, plus manual overrides | Explicit navigation list defined in one place |
| Blog support | Built-in blog with authors, tags, reading time, and feeds | Not included by default |
| Custom pages | Can include standalone custom pages outside the main docs | Pages generally follow the same documentation layout |
| Search | Includes built-in site search | Includes search with suggestions and highlighted matches |

Both tools convert Markdown and other written content into a polished, searchable website. Docusaurus is more flexible for mixed content like blogs and custom landing pages. MkDocs Material excels at clearly organized documentation with an explicit structure.

---

## Content Types in Docusaurus

A Docusaurus site typically has three kinds of content:

### 1. Documentation Articles

These are the main instructional pages. They are written in Markdown or an extended version of Markdown that can include interactive elements. Documentation articles are shown in a sidebar, which gives readers a sense of structure and lets them jump between related pages.

The sidebar is normally generated automatically from the content folder structure. In this project, the **Tutorial** section of the navigation comes from a sidebar that is generated automatically. This means that when a new documentation article is added or moved, the sidebar updates to reflect that change. It is also possible to define a sidebar manually, but the automatic approach is used here.

### 2. Blog Posts

Blog posts are also written in Markdown. They are different from documentation articles because they are organized by date and are intended for announcements, updates, tips, or community-oriented content.

A blog post can include:

- A **title**
- A **URL slug**, which controls the address of the post
- **Authors**, identified by short names that map to author profiles
- **Tags** for categorizing posts

For example, the welcome blog post included in this project has:

- Title: **Welcome**
- URL slug: **welcome**
- Authors: **slorber** and **yangshun**
- Tags: **facebook**, **hello**, **docusaurus**

The blog section also shows an estimated reading time for each post and provides RSS and Atom feeds so readers can subscribe. Blog posts can be split so that the beginning appears in a list view and the full content appears after clicking through. This is controlled by a truncation marker within the post.

### 3. Custom Standalone Pages

Docusaurus also supports custom pages that are built specifically for a particular purpose, such as a homepage, a landing page, or an interactive tool. These pages are not part of the documentation sidebar or the blog archive. They are built separately and can use more advanced layouts than a typical text page.

For most readers, custom pages are just additional pages on the site. From a content perspective, they are separate from the day-to-day writing of docs and blog posts.

---

## The Role of Sidebars

A sidebar is the navigation area that appears next to documentation articles. It shows the list of pages in a logical order and often includes expandable categories. Each documentation page appears in the sidebar, so readers always know where they are and what comes next.

In this project, the **Tutorial** sidebar is generated automatically from the documentation content. That means the sidebar order follows the structure of the documentation content itself. If you create a new documentation article and place it in the appropriate area, it will appear in the sidebar automatically. You can still override the order manually if needed.

Sidebars also provide **previous** and **next** navigation at the bottom of each documentation page, helping readers move through a tutorial or guide without returning to the menu.

---

## Front Matter

Front matter is a block of information at the top of a content file that provides metadata about that page. It is not shown as part of the main page content, but it controls how the page is displayed and organized.

Typical metadata includes:

- The page title
- The URL slug
- Authors of a blog post
- Tags for categorization
- Draft status, if the page should not yet be published
- Sidebar position or order

Front matter gives authors a simple way to set these properties without needing to alter the visible text of the page.

---

## Tags and Authors

**Tags** are short labels applied to blog posts. They help readers find related content. A post might be tagged with topics like `facebook`, `hello`, or `docusaurus`, as in the welcome post example. Docusaurus automatically creates tag-based views that collect all posts sharing a tag.

**Authors** are identifiers that associate a blog post with a particular person. Docusaurus keeps author information separate from the blog post itself. The post references an author by a short name, and the system displays the author’s name, avatar, and other details on the published page. This project includes authors named `slorber` and `yangshun` in the welcome post.

The platform can warn you if a blog post uses an inline tag or author that has not been declared properly, helping keep author and tag usage consistent.

---

## How MkDocs Material Navigation Maps Content

MkDocs Material uses a different approach from Docusaurus. Instead of automatically generating navigation from folder structure, the navigation is defined explicitly in one place. This allows precise control over the order and grouping of pages.

In this project’s MkDocs Material site, the navigation is:

- **Welcome**
- **Core Integrations**
  - Bitwarden Deployment
  - Storage with CSI for S3
  - Observability via OpenLIT
  - Database Cluster with Pangolin (CE)

Each entry in this list points to a specific Markdown document. Readers see these entries as a top navigation bar and/or a sidebar, depending on the theme configuration. The visible order always follows this explicit list, regardless of the underlying file names or folder layout.

This is useful when the content files are named in a way that is convenient for authors but you want the public navigation to use clean, human-readable labels.

---

## Build Output vs. Source Content

The content you write — Markdown files, images, and configuration — is the **source content**. It is what authors edit and review.

The website that readers visit is the **build output**. It is generated by the static site generator from the source content. Readers do not see the source files; they see the finished pages.

Deployment works like this:

1. An author writes or updates source content.
2. The static site generator builds a new version of the site.
3. The build output is copied to a hosting service.
4. Readers see the updated pages.

This separation means that changing a page is a deliberate process: edit the source, rebuild, and deploy. Nothing changes on the live site until the build output is replaced. It also means that the source content can be stored in a version-controlled environment while the public site remains stable until a new build is published.

---

## Summary

This platform uses two static site generators:

- **Docusaurus** for product documentation, blog posts, and custom pages. It provides automatic sidebars, front matter metadata, tags, authors, reading time, and RSS/Atom feeds.
- **MkDocs Material** for solution guides with an explicitly defined navigation order. It provides search features, code copy buttons, and clear navigation tabs.

Regardless of which generator is used, content starts as written source files and becomes a published site only after a build step produces the final output. Authors work with source content; readers interact with the published build.