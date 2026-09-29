# Your First Documentation Page - Quick Start

Create your first Markdown documentation page, understand front matter, and see changes appear instantly in the sidebar.

## Prerequisites

Before you begin, make sure you have:

- A Docusaurus site set up on your computer
- The development server running — open a terminal in your project and run `npm run start`

Keep that terminal window open throughout this guide. The development server is what makes your changes appear automatically.

## Understand the docs folder

Your documentation pages live in one dedicated place: the `docs` folder at the root of your project. Every Markdown file you place inside this folder becomes a page on your site and appears in the sidebar automatically.

When you first set up your site, Docusaurus adds a few sample pages to the `docs` folder so you can explore and modify them right away.

## Step 1: Edit an existing page

The quickest way to learn the content workflow is to change something that already exists.

1. Open the `docs` folder in your file explorer or code editor.
2. Look inside for a file like `intro.md` from the starter template.
3. Find the heading near the top of the file — it looks like `# Introduction`.
4. Change the heading text, for example to `# Welcome to My Site`.
5. Save the file.

Now switch to your browser. The page you were viewing updates on its own, without you restarting anything or manually refreshing.

## Step 2: Understand front matter

At the very top of each documentation file, you'll see a small block enclosed by three dashes on the opening and closing lines. This is called **front matter**, and it holds settings that control how your page behaves.

Here is a simple front matter example:

```
---
title: My First Page
sidebar_position: 1
---
```

The two settings shown here are the most common ones to start with:

- **`title`** — the label displayed in the sidebar and in the browser tab for this page.
- **`sidebar_position`** — where the page sits in the sidebar. The number `1` places the page first.

Everything after the closing three dashes is regular page content written in Markdown.

## Step 3: Add a new page

Now create a brand-new page from scratch to see the full workflow.

1. In the `docs` folder, create a new file and name it `hello.md`.
2. Add front matter at the top of the file:

```
---
title: Hello World
sidebar_position: 2
---
```

3. Below the front matter, write your page content in Markdown. For example:

```
# Hello World

This is my first documentation page. Welcome!

## What I learned

I can add a page by creating a Markdown file in the docs folder.
```

4. Save the file.

Switch to your browser. Your new page now appears in the sidebar under the label **Hello World**, positioned below your first page based on its `sidebar_position` value.

## Markdown basics you can use

Docusaurus renders standard Markdown, so the formatting you already know works as expected:

- Use `#`, `##`, and `###` for headings of different sizes
- Use `**bold text**` for emphasis
- Use `-` or `*` to create bullet lists
- Wrap links like `[link text](https://example.com)`

A heading that matches your `title` helps keep the page preview consistent, but your `title` is what drives the sidebar label.

## See changes hot-reload

The development server watches your files while it runs. Every time you save a Markdown file, the browser reflects the change within moments — no manual refresh needed.

Try it now: add a new sentence to `hello.md`, save the file, and watch the page update itself in the browser.

## Next steps

You now understand the basic content workflow: place Markdown files in the `docs` folder, set the title and sidebar position with front matter, and let the development server handle the rest.

When you're ready to go further, explore these topics:

- Organizing pages into folders and categories for a larger sidebar
- Adding images, links, and lists to make pages more useful
- Using additional front matter options to customize titles, descriptions, and sidebar behavior