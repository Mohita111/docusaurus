# Understanding Docusaurus Build and Runtime Workflow

When you create content for a Docusaurus site, the journey from your writing to a live page follows a consistent lifecycle. This guide explains each stage in plain language and helps you spot issues when something goes wrong.

## What happens behind the scenes

### 1. Content is loaded
Docusaurus looks at your Markdown and MDX files. Markdown is for everyday pages and blog posts. MDX extends Markdown so you can include interactive elements directly in a document.

### 2. Plugins prepare the content
Plugins add structure and metadata to your content. They can create page addresses, menus, sidebars, and relationships between pages.

### 3. Page addresses are generated
Every content item gets a page address. Internal links use these addresses, so visitors can move around quickly without waiting for a full new page load.

### 4. Static HTML is produced
In a production build, Docusaurus pre-renders each page as HTML. This helps visitors see content quickly and helps search engines read your site.

### 5. The page becomes interactive in the browser
After the first HTML appears, the site attaches interactive behavior. From then on, navigation and updates happen directly in the visitor's browser.

## Development server and hot reload

In development:
- You start a local preview.
- You edit a document or setting.
- Docusaurus detects the change and updates the preview automatically.
- Changes appear quickly, usually without a full browser refresh.

This is a working preview, not the final site.

## Production build

When you are ready to publish:
- Docusaurus creates a clean, optimized version of the site.
- It transforms all content into static, browser-ready pages.
- It applies metadata and site details.
- The result is a set of files you can host with any static web host.

The production build catches errors that a live preview may accept, so a site can work in development but fail in production.

## How Markdown and MDX become pages

When a document is processed:
- The text is converted into a format a browser can display.
- Headings are recognized and turned into a table of contents.
- The first main heading is identified as the page title.
- Images and other attached assets are collected so they display correctly.
- With MDX, interactive elements become part of the page just like regular content.

This means you can write mostly plain text and still produce a rich, interactive page.

## How routing works for visitors

Docusaurus automatically maps content to page addresses. When someone visits your site:
- The site displays the correct page for the address they requested.
- Internal navigation updates the visible page without a full server round trip.
- Docusaurus normalizes addresses so minor differences do not break navigation.
- If the site's base address is not configured correctly, a warning banner appears.

This keeps navigation smooth and helps you catch setup problems early.

## The application shell

Every rendered page sits inside a shared shell:
- Site metadata is applied consistently.
- The active theme controls appearance.
- A safety layer catches unexpected display errors and shows a helpful surface instead of a blank page.
- Navigation waits for the next page to be ready before updating the screen.

This is why your pages keep a consistent look and remain usable even when a single page has a problem.

## Debugging build issues

Use this checklist when the preview or build does not work as expected.

1. Check the development preview first. It often shows live errors with more context.
2. Run a production build. A clean build can reveal problems that a live preview hides.
3. Review your content files. Look for malformed front matter, broken links, or unsupported interactive syntax.
4. Check the site address settings. A misconfigured base address can cause missing pages or broken navigation.
5. Clear cached output. Old generated files can sometimes cause stale content.
6. Verify headings and titles. If the table of contents or page title is missing, the document headings may not be structured as expected.
7. Confirm that images and attached files are referenced correctly.

If a page fails after build, the issue may be in a single document or interactive element rather than the whole site. Simplify the page to isolate the problem.

## Summary

Docusaurus moves your content through a predictable pipeline:

load content → run plugins → generate page addresses → pre-render static pages → make the site interactive.

Knowing this flow helps you understand why a site behaves as it does and makes build problems easier to locate.