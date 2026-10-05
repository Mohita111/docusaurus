# How to write markdown content with front matter

Front matter sits at the very top of a Markdown or MDX file and contains metadata about the page, such as its title and sidebar order. This guide shows you how to add front matter to a new content page.

## Prerequisites

Before you begin, make sure you have:

- Access to a Docusaurus site
- Permission to create or edit`.md` or`.mdx` files
- A text editor or a browser-based file editor

## Add front matter to a new markdown file

1. Create a new`.md` or`.mdx` file in the folder where your content lives.
2. At the very top of the file, type three dashes (`---`) on their own line.
3. Add your page settings as key-value pairs in YAML format, with one setting per line.
4. Close the front matter with three more dashes (`---`) on their own line.
5. Below the closing dashes, write the page content in Markdown.

Here is a minimal example:

```markdown
---
title: My first guide
---

This is the content of my page.
```

The`title` value becomes the page heading, and the text below the closing dashes becomes the page body.

## Add common front matter fields

You can control more than just the title. Common fields include:

-`title`: the page heading and browser tab label
-`description`: a short summary that may appear in search results and social media previews
-`sidebar_position`: the numeric order of the page in the sidebar (lower numbers appear first)
-`tags`: one or more labels that group and filter related pages
-`draft`: set to`true` to hide the page from the published site
-`slug`: a custom URL path for the page (if not set, derived from the file name)

Example with several fields:

```markdown
---
title: Create a project
description: Learn how to start a new project from scratch
sidebar_position: 1
tags:
  - getting-started
  - projects
draft: false
---

# Create a project

Use the instructions below to set up your first project.

## Step one
```

## Format requirements for front matter

Front matter must follow YAML syntax rules. Here are key formatting requirements:

- **No blank lines before the opening dashes**: Front matter must start at line 1 of the file.
- **Proper indentation for lists**: Use consistent spaces (not tabs) when listing multiple values. Each list item should be on a new line, preceded by a hyphen and a space.
- **Colons and spacing**: Every field name must be followed by a colon and a space. For example,`title: My Page` is correct;`title:My Page` is not.
- **String values with special characters**: Wrap values in quotes if they contain colons, dashes, or other special characters. For example,`description: "Follow these steps: create, test, deploy"`.

## Expected result

After you save the file, the page appears on the site with:

- The title you set as the page heading
- The description available for search previews and social sharing
- The sidebar item positioned according to`sidebar_position`
- Tags applied for filtering and organization

If the site has a live development server running, the page appears automatically after you save the file. If you're building the site for production, run the build command and check the output directory.

## Common issues

### The page shows no title or uses a fallback title

Make sure the front matter is at the very top of the file with no blank lines before the first three dashes. Also confirm that the`title` field is spelled correctly and has a space after the colon.

### The page does not appear in the sidebar

Check that the`sidebar_position` field is present and set to a number. If the field is missing, the page may still appear but in default order. If the page is missing entirely, make sure the file is in a folder that the site includes in the sidebar.

### The page shows a build error or displays incorrectly

Look for syntax issues in the front matter:

- **Missing closing dashes**: Every front matter block must end with`---` on its own line.
- **Incorrect indentation for list values**: Tags and other lists must use spaces (not tabs) with each item on a new line, preceded by a hyphen.
- **Missing colon or space after field name**: Write`title: ` not`title:` or`title :`.
- **Tabs instead of spaces**: YAML requires spaces for indentation. Check your editor settings.
- **Unquoted values with special characters**: Values containing colons, quotes, or leading spaces must be wrapped in quotes.

### A field behaves unexpectedly or is ignored

Some fields only apply to specific content types. For example:

-`sidebar_position` controls ordering in documentation but not in blog posts.
-`tags` work in documentation and blog content but may not apply to other content types.
-`slug` overrides the URL path for documentation pages.

Use only the fields that are supported for your content type. Refer to the MDX & Markdown Authoring guide for your content type's supported fields.

### The page content appears but front matter fields do not take effect

Verify that the field names match your site's schema exactly. Some sites have custom fields beyond the standard ones. Check your site administrator's documentation or the configuration file for the list of supported fields.

## Related guides

- Embed interactive React components in MDX
- Render Mermaid diagrams
- Optimize and transform images
