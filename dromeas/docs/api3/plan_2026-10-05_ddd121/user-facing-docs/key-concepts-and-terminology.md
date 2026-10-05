# Key concepts and terminology

Welcome! This guide explains the core ideas you'll encounter as you build and customize a Docusaurus site. Use it as a reference whenever you come across an unfamiliar term in the rest of the documentation.

## Core concepts

### Site

Your complete web project. A site includes all of your pages, blog posts, documents, navigation, visual design, and configuration. When you run a build, everything comes together into a single site that you can publish.

### Route

The web address where a page or document is available. For example, your homepage, a documentation article, or a blog post each have their own route. Visitors reach content through routes.

### Plugin

An add-on that extends what your site can do. Plugins can create new pages, organize content, modify how the site is built, or connect to outside services. You choose which plugins to include and adjust their settings to match your needs. Docusaurus comes with several built-in plugins you can enable, including the docs plugin, blog plugin, and pages plugin.

### Theme

The visual layer of your site. A theme supplies the layout, styling, navigation elements, and other user interface pieces that determine how your site looks. The default Docusaurus theme is called "classic." Themes are designed so you can customize parts of them while still receiving future improvements from the theme's maintainers.

### Preset

A convenient bundle of plugins and themes that you install together. Instead of adding each piece separately, you can include a preset and get a typical, well-matched setup right away. The default Docusaurus setup uses a preset to combine the documentation, blog, and pages features with the classic theme.

### Content plugin

A type of plugin that manages a particular kind of content. A content plugin handles finding, organizing, and displaying material such as documentation articles, blog posts, or standalone pages. The docs plugin, blog plugin, and pages plugin are all examples of content plugins.

### Sidebar

The navigational list that appears next to your documentation. Sidebars control which documents are displayed and in what order. You can group documents, create categories, and generate part of the sidebar automatically based on your document titles and file names.

### Front matter

A special section at the top of a Markdown or MDX document where you set information about that document. You might define a title, a description, custom sidebar text, or other options. Front matter is enclosed between two lines of three dashes.

### MDX

An extension of Markdown that lets you include interactive elements built with React directly inside your content. With MDX, you can mix regular formatted text with dynamic features such as buttons, charts, or custom layouts.

### Static asset

A file that is served exactly as it is, without any processing. Images, downloadable files, fonts, and similar resources are examples of static assets. You add them to your site and Docusaurus includes them in the final output unchanged.

### Build output

The final collection of files produced when you build your site. After a successful build, these are the files you upload to your hosting provider. The build output contains all the pages, scripts, styles, and assets needed to run your published site.

### Swizzling

A way to customize part of a theme safely. When you swizzle a theme element, you make a working copy of it that you can edit. Your copy takes priority, so your changes appear on the site. Because the original theme stays intact, you can still update the theme later without automatically losing your customizations.

### Versioning

The practice of maintaining multiple documentation versions so readers can access docs for different software releases. With versioning, you can keep documentation for older versions available while publishing new versions as your product evolves. Docusaurus handles version management automatically, allowing you to organize docs by version and provide version selectors to visitors.

## Choosing between docs, blog, and pages

Docusaurus includes three content plugins, and each one is designed for a different job. Understanding their roles helps you decide where to put your content.

### Docs

The docs plugin builds a structured documentation area. It organizes Markdown and MDX articles into sections, supports multiple versions of your documentation, and provides sidebars for navigation. Use docs when you need a reference manual, user guide, or any collection of articles that readers browse in a defined order. The docs plugin stores your content in markdown and MDX files organized in a docs directory, making them searchable and versionable alongside the rest of your site.

### Blog

The blog plugin creates a time-based list of posts. Posts appear in reverse chronological order, and you can add tags and author information. The blog plugin also generates archive pages, pagination, and individual post pages. Use blog for news, announcements, release notes, and other content where the newest item is the most important. Blog posts support front matter metadata including author names and dates.

### Pages

The pages plugin turns standalone pieces of content into individual pages that are not tied to docs or blog. Use pages for one-off content like a homepage, an about page, a pricing page, or a landing page for a campaign. Pages have full flexibility but do not automatically receive sidebars or chronological organization.

## Understanding swizzling in practice

Swizzling gives you a controlled way to change the appearance or behavior of your site's theme. Instead of editing the theme directly, you copy a specific theme element and modify the copy.

For example, if you want to change how the footer looks, you can swizzle the footer element. Docusaurus makes a copy for you, and from then on your copy is used on the site. You can edit the copy to add your own text, links, or layout.

Swizzling is useful because it keeps your changes separate from the theme. When the theme releases an update, you can incorporate that update without having to redo all of your work. You also have control over which elements you swizzle, so you can customize as little or as much as you need.

## Glossary

| Term | What it means |
| --- | --- |
| **Blog** | A chronological collection of posts with author metadata and pagination, managed by the blog plugin |
| **Build output** | The final files produced when you build your site, ready for hosting |
| **Content plugin** | A plugin that manages a specific kind of content, such as docs, blog, or pages |
| **Docs** | A structured, versionable documentation area organized in markdown and MDX files, managed by the docs plugin |
| **Front matter** | Metadata at the top of a Markdown or MDX document, enclosed in three dashes |
| **MDX** | Markdown extended to allow interactive React elements within your content |
| **Pages** | Standalone pages not tied to docs or blog |
| **Plugin** | An add-on that extends your site's capabilities |
| **Preset** | A bundled collection of plugins and themes that you install together |
| **Route** | The web address where a page or document is available |
| **Sidebar** | The navigational list that appears next to your documentation |
| **Site** | Your complete web project, including all content, design, and configuration |
| **Static asset** | A file served unchanged, such as an image or downloadable file |
| **Swizzling** | Copying and customizing a theme element to safely change your site's appearance or behavior |
| **Theme** | The visual layer that controls how your site looks; the default theme is "classic" |
| **Versioning** | Maintaining multiple documentation versions for different software releases |

## Related topics

- Features - Documentation Plugin to learn how docs are organized and structured
- Features - Blog Plugin to understand blog post management and metadata
- Features - MDX Content Loading for details on MDX authoring capabilities
