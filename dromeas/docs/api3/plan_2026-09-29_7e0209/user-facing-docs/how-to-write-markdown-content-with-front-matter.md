# How to write Markdown content with front matter

Front matter sits at the very top of a Markdown file and contains information about the page, such as its title and its order in the sidebar. This guide shows you how to add front matter to a new content page.

## Prerequisites

Before you begin, make sure you have:

- Access to a Docusaurus site
- Permission to create or edit `.md` or `.mdx` files
- A text editor or a browser-based file editor

## Add front matter to a new Markdown file

1. Create a new `.md` or `.mdx` file in the folder where your content lives.
2. At the very top of the file, type three dashes on their own line.
3. Add your page settings as key-value pairs, with one setting per line.
4. Close the front matter with three more dashes on their own line.
5. Below the closing dashes, write the page content in Markdown.

Here is a minimal example:

```markdown
---
title: My first guide
---

This is the content of my page.
```

The `title` value becomes the page heading, and the text below the closing dashes becomes the page body.

## Add common front matter fields

You can control more than just the title. Common fields include:

- `title`: the page heading
- `description`: a short summary that may appear in search results
- `sidebar_position`: the order of the page in the sidebar
- `tags`: one or more labels that group related pages
- `draft`: set to `true` to keep the page hidden

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

## Expected result

After you save the file, the page appears on the site with:

- The title you set as the page heading
- The description available for search previews
- The sidebar item in the position you specified
- Tags applied for filtering

If the site has a live development preview running, the page appears automatically after you save the file.

## Common issues

### The page shows no title or uses a fallback title

Make sure the front matter is at the very top of the file, with no blank lines before the first three dashes. Also confirm that the `title` field is spelled correctly and has no extra spaces before it.

### The page does not appear in the sidebar

Check the `sidebar_position` value. If it is missing, the page may still appear but in a default order. If the page is missing entirely, make sure the file is in a folder that the site displays in the sidebar.

### The page shows a build error or displays incorrectly

Look for syntax issues in the front matter, such as:

- A missing closing set of three dashes
- Incorrect indentation for list values like `tags`
- A missing colon after a field name
- Using tabs instead of spaces

### A field behaves unexpectedly

Some fields only affect certain types of content. For example, `sidebar_position` works for documentation pages but not for blog posts. Use only the fields that are supported for the type of content you are creating.