# Your first documentation page

Create your first documentation page by editing an existing Markdown file, then add a brand-new page and watch it appear in the sidebar automatically.

Docusaurus turns Markdown files into polished documentation pages. You don't need to know any programming to get started — just type text and save.

## Before you begin

Make sure your Docusaurus site is running locally. If it isn't, open a terminal, navigate to your website folder, and run:

```bash
npm run start
```

Leave that terminal window open while you work. It powers the live preview.

## Understand how documentation pages work

Every documentation page has two parts:

- **Front matter** — a short block at the very top of the file that controls the page title and where it appears in the sidebar.
- **Markdown content** — the actual text, headings, lists, and links that readers see.

Front matter looks like this:

```markdown
---
sidebar_position: 1
title: Hello
---

# Hello

This is my first page.
```

The `title` becomes the page heading and the sidebar label. The `sidebar_position` controls the order of pages in the sidebar — smaller numbers appear higher up.

## Edit an existing page

1. In your site files, open the **docs** folder.
2. Open one of the existing `.md` files in your text editor.
3. Change the title or add a new sentence below the front matter.
4. Save the file.

Your browser updates automatically. You don't need to refresh the page — Docusaurus detects the change and reloads the preview for you.

## Add a new page

1. In the **docs** folder, create a new file and give it a name ending in `.md`, such as `hello.md`.
2. Add front matter at the very top of the file:

   ```markdown
   ---
   sidebar_position: 2
   title: Hello
   ---
   ```

3. Below the front matter, write your content using Markdown:

   ```markdown
   # Hello

   Welcome to my new page.

   ## What you'll learn

   - How to create a page
   - How to order pages in the sidebar
   - How to format text

   **Bold text** and [links](https://example.com) work too.
   ```

4. Save the file.

The new page appears in the sidebar immediately, in the position you set with `sidebar_position`. Click it to see your content.

## Use Markdown formatting

You can format your page with everyday Markdown:

- `#` starts a top-level heading
- `##` starts a subheading
- `-` creates a bulleted list
- `**text**` makes text bold
- `[link text](https://example.com)` creates a link

Experiment with these and watch the preview update as you type.

## Next steps

Now that you can create simple pages, try reordering existing pages by changing their `sidebar_position` values, or add several new pages to build out a small documentation section.