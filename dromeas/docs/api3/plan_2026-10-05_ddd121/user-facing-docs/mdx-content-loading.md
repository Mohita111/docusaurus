# MDX content loading

MDX content loading is Docusaurus's system for transforming your markdown and MDX files into interactive documentation. It combines the simplicity of markdown with the power of React components, letting you write documentation that feels natural while embedding interactive elements directly in your content.

## What is MDX content loading and who it's for

MDX is an extension of markdown that lets you use React components alongside regular prose. The MDX content loading feature processes your markdown and MDX files, extracting metadata, generating navigation structures, and rendering both static content and interactive components in a unified experience.

This feature is designed for:

- **Content authors** who want to write documentation in familiar markdown syntax
- **Technical writers** who need to include interactive examples or visualizations
- **Developers** building documentation sites with reusable UI components
- **Documentation teams** managing large content libraries with consistent formatting

## What it enables

The MDX content loading system handles the complete transformation pipeline:

### Processing your content files

When you create a markdown or MDX file, the loader reads it and extracts its components:

- **Frontmatter metadata**: Title, publication date, authors, and custom fields you define
- **Heading structure**: Automatically generates table of contents from your document's headings
- **Prose and components**: Combines regular markdown text with embedded React components
- **Code blocks**: Processes code examples with syntax highlighting and optional line numbers

### Transforming markdown syntax

The loader extends standard markdown with additional capabilities through remark plugins:

- **GitHub Flavored Markdown (GFM)**: Tables, strikethrough, task lists, and other GitHub-style syntax
- **Frontmatter support**: YAML metadata at the top of your files
- **Emoji**: Automatic emoji shortcode conversion (`:smile:` becomes )
- **Custom directives**: Extensible syntax for custom blocks and components
- **Raw HTML**: Preserves and processes embedded HTML alongside markdown

### Enabling interactive content

MDX lets you write React components directly in your documentation:

```
# My Guide

This is regular markdown with a chart:

<MyChart data={chartData} />

You can mix components and prose freely.
```

The loader processes these components so they render alongside your documentation text, enabling interactive examples, embedded tools, and dynamic content.

### Extracting and organizing structure

The system automatically identifies:

- Document headings and their nesting levels
- Metadata from frontmatter fields
- Content sections for building navigation structures
- Title information for page headers and sidebars

## Key options and configuration

### Custom markdown processing

You can extend the markdown parser by configuring remark plugins. These plugins process your markdown syntax before it's converted to HTML:

- **Built-in plugins** like GFM, emoji, and directives come enabled by default
- **Custom remark plugins** allow you to add domain-specific syntax or transformations
- **Plugin ordering** matters—earlier plugins process content before later ones

### Code block customization

The loader processes code blocks with options for:

- **Syntax highlighting**: Automatic detection and coloring based on language
- **Line numbers**: Optional line number display for code examples
- **Line highlighting**: Mark specific lines to draw attention
- **Language labels**: Display the code language as a badge or label

### Heading extraction configuration

Control how the system generates table of contents:

- **Heading levels**: Choose which heading levels appear in navigation (typically h2 and h3)
- **Custom anchors**: Override automatic anchor generation for headings
- **Metadata integration**: Extract heading structure programmatically for custom layouts

## How it works: The processing pipeline

The MDX content loading system follows a predictable flow:

1. **Read the file**: Load your`.md` or`.mdx` file from disk
2. **Parse frontmatter**: Extract YAML metadata from the top of the file
3. **Process with remark**: Transform markdown syntax using configured remark plugins (GFM, emoji, directives.)
4. **Convert to MDX AST**: Create an abstract syntax tree that understands both markdown and JSX
5. **Enhance with rehype**: Process HTML-level transformations (raw HTML handling, syntax highlighting)
6. **Extract structure**: Identify headings, generate table of contents, extract metadata
7. **Output JavaScript**: Generate executable JavaScript that renders your content

This pipeline ensures your content is transformed consistently while preserving both markdown simplicity and React interactivity.

## Benefits and use cases

**Combine markdown's simplicity with React's power**: Write in familiar markdown syntax while having the full capabilities of React components when you need interactivity.

**Reuse UI components across documentation**: Define button styles, callout boxes, or complex layouts once as React components, then use them everywhere in your documentation without repeating HTML.

**Extend markdown syntax**: Use remark plugins to add custom blocks (like admonitions, code tabs, or info boxes) that aren't part of standard markdown.

**Maintain consistency**: Centralized configuration for code highlighting, heading extraction, and metadata handling ensures all your content follows the same rules.

**Improve perceived performance**: The system works with image optimization features to load images efficiently and display placeholders while full images load.

## Key capabilities and limits

### What works well

- **Markdown-first authoring**: Write most content in plain markdown, add components where needed
- **Complex metadata**: Extract and use custom frontmatter fields for authors, dates, categories, and more
- **Dynamic content**: Embed React hooks and state in your documentation for truly interactive examples
- **Plugin extensibility**: The remark and rehype plugin systems let you customize syntax and processing
- **Performance-optimized**: Static generation means fast page loads and excellent SEO

### Important boundaries

- **Requires markdown knowledge**: You need to understand basic markdown syntax to write content effectively
- **React skills needed for components**: Embedding custom React components requires JavaScript and React familiarity
- **Plugin complexity**: Advanced remark or rehype plugins can be challenging to develop and debug
- **MDX parsing overhead**: Complex MDX with many embedded components can be slower to parse than plain markdown
- **Maintenance complexity**: Mixing markdown and JSX can make content harder to maintain if overused—balance is important

## Evidence of functionality

The MDX content loading feature is powered by the`@docusaurus/mdx-loader` package, which processes all your content files. Key capabilities are provided by:

- **`@mdx-js/mdx` (v3.1.1)**: The core MDX compiler that transforms JSX alongside markdown
- **`remark-frontmatter` (v5.0.0)**: Extracts YAML metadata from the top of files
- **`remark-gfm` (v4.0.1)**: Adds GitHub Flavored Markdown syntax support
- **`remark-emoji` (v5.0.2)**: Converts emoji shortcodes to actual emoji
- **`remark-directive` (v4.0.0)**: Enables custom markdown directives for extensibility
- **`rehype-raw` (v7.0.0)**: Processes raw HTML embedded in your content
- **`unified` (v11.0.5)**: The plugin system that coordinates all transformations

The loader integrates with Docusaurus's utilities for frontmatter parsing (via`@docusaurus/utils`) and validates configurations using`@docusaurus/utils-validation`.

---

**Related topics:**
- MDX & Markdown Authoring
- Documentation Plugin
- Build Bundling and Optimization
