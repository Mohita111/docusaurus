# Pages

Docusaurus sites can include standalone pages for content that isn't part of your main documentation or blog. Use pages for custom landing pages, about pages, contact pages, and any other independent content you want to share.

## What you can do with pages

- Create a separate page for any purpose you choose
- Write page content using MDX, which combines Markdown with React features
- Add interactive React components such as forms, maps, or animations
- Give each page its own web address based on the page name
- Include page metadata, such as a preview image, at the top of an MDX page

## How pages work

When you add a page file to your site's pages area, Docusaurus detects it automatically and creates a route for it. The page appears under its own URL, and Docusaurus builds it ahead of time so visitors see it quickly.

MDX pages support common content options, including:

- Admonitions such as notes, tips, and warnings
- Additional Markdown and MDX parsing options
- Links that resolve to other pages on your site

If you use a React page, you can define the full layout and include any interactive components you need.

## Boundaries

Standalone pages don't receive sidebars automatically like documentation pages do. If you want visitors to find a page easily, add links to it from your existing navigation, documentation, blog, or other pages.

Pages are best for content that stands alone. For longer documentation with organized sections, use the documentation features instead.