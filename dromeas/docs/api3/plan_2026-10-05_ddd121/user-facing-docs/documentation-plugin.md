# Documentation plugin

The Documentation Plugin organizes and structures your product documentation into a navigable, versioned site. It automatically discovers markdown files, generates sidebar navigation, manages multiple documentation versions, and enables readers to browse, search, and stay updated with your docs.

## What it does

The Documentation Plugin powers the core documentation experience on your Docusaurus site. It reads your markdown files from a designated docs directory, automatically extracts metadata (like titles and descriptions), generates a sidebar menu based on your folder structure, and makes each document discoverable through search and navigation. You can maintain multiple versions of your documentation—current and archived—and readers can switch between them. The plugin also tracks document metadata, generates edit links so readers can contribute improvements, and manages how documents appear in breadcrumbs, pagination, and search results.

## Who it's for

**Documentation teams** maintaining product documentation, API references, guides, and knowledge bases. **Site administrators** managing documentation structure and versions. **Content authors** publishing and updating docs. **Site visitors** browsing and searching documentation content.

## Key capabilities

### Automatic sidebar generation

The plugin discovers all markdown files in your docs directory and automatically organizes them into a sidebar menu. It respects your folder structure—nested folders become nested menu categories—and uses document titles and metadata to create readable labels. You can also create an explicit sidebar configuration file to customize the structure beyond what the folder hierarchy provides. Files are sorted by alphabetical order or by a sidebar position number you add to the front matter (the metadata section at the top of each markdown file).

### Metadata extraction and front matter

Each document's front matter defines properties like the title, description, sidebar label, sidebar position, custom edit URL, and pagination links. The plugin validates this metadata and uses it to generate proper page titles, search indexes, and navigation. You can mark documents as drafts or unlisted so they're excluded from production builds or search results. You can also customize the document's slug (the URL part) or assign it a custom ID.

### Multiple documentation versions

You can maintain current and archived versions of your documentation side by side. Each version has its own docs directory and sidebar configuration. Readers see a version selector that lets them switch between versions without leaving the site. This is essential for products with multiple active releases.

### Edit links and contributor pathways

The plugin automatically generates "Edit this page" links that point to your source repository (GitHub, GitLab, or others). This lowers the barrier for community contributions and makes it easy for readers to suggest improvements. You can configure edit URLs per provider or use a custom function to generate them dynamically.

### Navigation and pagination

The plugin automatically adds previous/next links between documents based on the sidebar order or explicit front matter configuration. Each document receives its sidebar context, which powers breadcrumbs and navigation panels. Unlisted documents are excluded from pagination and search so you can draft or review documents privately.

### Search and tagging

Documents are indexed for search with their full content, title, and description. You can organize documents with tags in the front matter, and the plugin creates tag index pages so readers can browse posts by topic. Tags are normalized and validated.

### Last update tracking

The plugin tracks when each document was last modified—from your Git history or from the front matter—and displays this information on the page. You can show both the last update date and the author who made the change.

## Configuration approaches

You choose how to organize your sidebar: use **conventional automatic generation** where the plugin creates the sidebar from your folder structure, or use **explicit sidebar configuration** in a`sidebars.js` file for fine-grained control. Automatic generation is faster to set up but less flexible. Explicit configuration gives you full control over labels, nesting, and ordering.

For versioning, decide whether to maintain one current version or multiple active versions. Configure your edit URL provider (GitHub, GitLab, Gitea, or a custom function) and decide whether to edit localized or source files.

## Typical structure

A standard docs folder looks like this:

```
docs/
  ├─ getting-started/
  │  ├─ installation.md
  │  └─ quick-start.md
  ├─ guides/
  │  ├─ authentication.md
  │  └─ api-usage.md
  └─ api/
     ├─ rest-endpoints.md
     └─ webhooks.md
```

The plugin discovers these files, extracts their front matter, and organizes them by directory. If you add a sidebar configuration file, you can customize how they're grouped and labeled.

## What happens during build

When you build your site, the plugin:

1. Discovers all markdown and MDX files in the docs directory
2. Reads and validates the front matter of each file
3. Generates page slugs (URLs) and extracts titles and descriptions
4. Processes metadata like last update dates and tags
5. Generates or loads the sidebar configuration
6. Builds navigation links between documents
7. Indexes documents for search
8. Creates version metadata if using multiple versions

The result is a structured set of pages with navigation, metadata, and search integration—all ready to render.

## Key boundaries and considerations

**Hierarchical structures work best.** The plugin is optimized for documentation organized in folders and subfolders. Deeply nested structures are supported but may become harder to navigate.

**Versioning requires planning.** If you maintain multiple versions, plan how you'll handle breaking changes and updates across versions. Each version has its own docs directory and sidebar configuration.

**Sidebar configuration is strict.** If you use explicit sidebar configuration, all document IDs must correspond to actual files, and the configuration is validated at build time.

**Unlisted documents are production-only.** You can mark documents as unlisted during development and have them appear in production, or vice versa, using the`unlisted` field in front matter.

**Large doc sites may need optimization.** Sites with hundreds of documents benefit from configuring search filters and using version management to keep the index manageable.

## Integration with other features

The Documentation Plugin works closely with static site generation to transform markdown into HTML, MDX content loading to enable interactive components in docs, and search functionality to make docs discoverable. It pairs with the Blog Plugin for time-ordered content like release notes, and with client redirects to gracefully handle documentation restructures and URL changes.
