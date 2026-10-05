# How to publish a new blog post

Publishing a blog post in Docusaurus involves creating a markdown or MDX file in your blog directory with the required front matter metadata. This guide walks you through the process step by step.

## Prerequisites

Before you begin, you need:

- A Docusaurus project with the blog plugin enabled
- Write access to your project's blog directory (typically`blog/`)
- Understanding of markdown or MDX syntax
- Optional: an authors map file if you want to reference authors by key

## Step-by-step instructions

### 1. Create your blog post file

Create a new`.md` or`.mdx` file in your blog directory. The filename determines several aspects of your post:

**Using a date-based filename (recommended):**

Name your file with the format`YYYY-MM-DD-post-title.md` or`YYYY-MM-DD/post-title.md`. For example:
-`blog/2024-01-15-my-first-post.md`
-`blog/2024-01-15/my-first-post.md`

The date in the filename becomes your post's publication date automatically.

**Using a simple filename:**

You can also use a simple name like`blog/my-post.md` if you prefer to specify the date in front matter (see step 2).

### 2. Add front matter to your post

At the very top of your file, add front matter in YAML format enclosed by`---` markers. Include at least a title:

```yaml
---
title: My Blog Post Title
description: A brief description of your post
date: 2024-01-15
---
```

**Common front matter fields:**

- **title** (required): The heading displayed for your post
- **description**: A short summary shown in blog listing pages and feeds
- **date**: Publication date (use format`YYYY-MM-DD` or`YYYY-MM-DD HH:MM:SS`); overrides the filename date if provided
- **authors**: Attribution for who wrote the post (see "Adding authors" below)
- **tags**: Comma-separated or array of keywords to categorize your post
- **slug**: Custom URL path for your post; auto-generated from filename if not specified
- **image**: URL to an image displayed with your post
- **draft**: Set to`true` to hide the post from published listings (won't appear on your site)
- **unlisted**: Set to`true` to make the post invisible to listings and feeds but still accessible via direct link

### 3. Add authors (optional)

Reference authors in your front matter using an authors map file. In your front matter, add:

```yaml
---
title: My Blog Post
authors: [alice, bob]
---
```

Or for a single author:

```yaml
---
title: My Blog Post
authors: alice
---
```

The author keys (like`alice` and`bob`) must match keys in your`authors.yml` file located in your blog directory. Each author key maps to author details like name, title, and profile image.

Alternatively, you can define authors inline without an authors map:

```yaml
---
title: My Blog Post
authors:
  - name: Alice Johnson
    title: Technical Writer
    url: https://example.com/alice
    image_url: https://example.com/alice.jpg
---
```

### 4. Write your blog post content

Below the closing`---`, write your post content in markdown or MDX:

```markdown
---
title: My First Post
---

This is my first paragraph.

## My First Section

Here's more content with **bold** and *italic* text.
```

If you're using MDX format (`.mdx` file extension), you can also embed interactive React components directly in your content.

### 5. Add a truncation marker (recommended)

To create a shorter preview on the blog listing page, add a truncation marker at the point where the preview should end:

```markdown
---
title: My Blog Post
---

This part appears in the preview.

<!-- truncate -->

This part only appears on the full post page.
```

Or if using MDX:

```mdx
---
title: My Blog Post
---

This part appears in the preview.

{/* truncate */}

This part only appears on the full post page.
```

Without a truncation marker, Docusaurus generates a warning during the build process.

### 6. Save and build

Save your file in the blog directory, then build your site:

```bash
npm run build
```

Or to preview locally before publishing:

```bash
npm run start
```

## Expected result

After successfully publishing, your post will:

- Appear on the main blog listing page in reverse chronological order (newest first)
- Be included in any tags feeds and RSS/Atom feeds if configured
- Have its own dedicated page at a URL based on the date and slug (for example:`/blog/2024/01/15/my-post`)
- Show metadata like publication date, reading time, and author information
- Appear in the sidebar under "Recent posts" (depending on sidebar configuration)

## Common issues and troubleshooting

**Post doesn't appear on the site**

Check that your front matter is valid YAML. Common issues include:
- Missing or mismatched`---` delimiters
- Incorrect indentation in the YAML section
- Special characters in the title that need quoting (e.g.,`title: "My Post: A Guide"`)

If your front matter is valid, verify the post isn't marked as`draft: true`.

**Author references cause errors**

If you receive an error about an author key not being found, check that:
- The author key in your front matter matches exactly with a key in your`authors.yml` file
- The`authors.yml` file exists at the path configured in your blog plugin settings (default:`blog/authors.yml`)
- The file is valid YAML format

**Post appears with wrong date**

The publication date comes from:
1. The`date` field in front matter (highest priority)
2. The date in the filename (if using date-based naming)
3. The file creation date (fallback if neither of the above is provided)

If your post has the wrong date, add or correct the`date` field in front matter.

**Build warnings about truncation markers**

If you see a warning during build about blog posts without truncation markers, add`<!-- truncate -->` or`{/* truncate */}` to your posts, or disable the warning by setting`onUntruncatedBlogPosts: 'ignore'` in your blog plugin configuration.

## Related resources

Blog Plugin, Learn more about blog features and configuration options.

MDX & Markdown Authoring, Explore advanced content authoring techniques including components and diagrams.

How to subscribe to RSS/Atom feed, Help readers stay updated with your latest posts.
