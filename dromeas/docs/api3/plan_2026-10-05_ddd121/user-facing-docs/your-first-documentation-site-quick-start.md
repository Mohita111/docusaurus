# Your first documentation site: Quick start

Get a working documentation site up and running in about 15 minutes. This guide walks you through creating a new project, adding content, running it locally, and building it for production.

## What you'll build

By the end of this guide, you'll have a documentation site that you can preview in your browser and share with others. The site will include a homepage, your first documentation page, and a custom site title.

## Prerequisites

Before you start, make sure you have Node.js version 24.21 or later installed on your computer. You can check your Node.js version by running:

```bash
node --version
```

If you don't have Node.js installed, download it from nodejs.org.

## Step 1: Create a new docusaurus project

Open your terminal and run the following command to scaffold a new documentation site:

```bash
npx create-docusaurus@latest my-docs classic
```

This command creates a new folder called`my-docs` with all the files you need for a documentation site. The`classic` template gives you a solid foundation with documentation, blog, and homepage features built in.

Once the command finishes, navigate into your new project folder:

```bash
cd my-docs
```

## Step 2: Explore the project structure

Your new project contains these key folders and files:

- **docs**: Where your documentation pages live (in Markdown and MDX format)
- **blog**: Where blog posts go
- **src**: Custom pages and components
- **docusaurus.config.js**: Your site's main configuration file (site title, navigation, plugins, and more)
- **sidebars.js**: Defines the structure of your documentation sidebar

Don't worry about understanding everything right now—you'll focus on the essentials to get running quickly.

## Step 4: Add your first documentation page

In the **docs** folder, you'll see an`intro.md` file. This is a documentation page written in Markdown format. Let's create a new page:

1. In your **docs** folder, create a new file called`my-first-page.md`
2. Add this content to the file:

```markdown
---
title: My First Page
---

# Welcome to my documentation

This is my first documentation page. I can write here in **Markdown** format.

## Why Docusaurus?

- Fast and modern documentation site
- Easy to write in Markdown
- Built-in search and mobile support

Here's a simple code example:

\`\`\`javascript
console.log("Hello, Docusaurus!");
\`\`\`
```

Save the file and check your browser—your new page should appear in the sidebar on the left, and you'll be able to click it to view it.

## Step 5: Customize your site title and metadata

Now let's personalize your site. Open`docusaurus.config.js` in your editor and find the following section near the top:

```javascript
const config = {
  title: 'My Site',
  tagline: 'Dinosaurs are cool',
  // ... rest of config
};
```

Change the title and tagline to match your project:

```javascript
const config = {
  title: 'My Project Docs',
  tagline: 'Everything you need to know',
  // ... rest of config
};
```

Save the file. Your browser will refresh automatically, and you'll see the new title at the top of the page and in the browser tab.

## Step 6: Build your site for production

When you're ready to share your documentation with the world, build it for production by running:

```bash
npm run build
```

This command compiles all your Markdown files, optimizes images and code, and creates a production-ready folder called`build`. This folder contains static HTML, CSS, and JavaScript files that you can deploy anywhere.

The build process may take a minute or two, depending on how much content you have. You'll see progress messages in your terminal.

## Next steps

You now have a working documentation site. Here are some things you can do next:

- Add more documentation pages in Markdown format to build out your content
- Customize your sidebar structure to organize your docs the way you want
- Publish a blog post to share updates with your audience
- Deploy your site to the web so others can access it
- Explore performance optimization to make your site even faster
