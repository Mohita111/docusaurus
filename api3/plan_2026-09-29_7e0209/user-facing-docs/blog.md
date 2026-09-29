# Blog

Blog gives you a place to publish updates, announcements, stories, and articles on your website. It is designed for teams that want to share news with their community, and for individuals who want to maintain a personal or project journal.

When you create a blog post, Docusaurus automatically gives it a dedicated page, a place in the blog listing, and navigation to the previous and next posts.

## Who the blog is for

The blog feature works well for several kinds of projects:

- Product teams that want to share release notes, changelogs, or company updates
- Open source maintainers who want to publish project news
- Personal websites and portfolios that include essays, tutorials, or regular updates
- Teams that need a simple publishing flow without maintaining a separate content platform

## What you can do

With the blog feature, you can:

- Write posts in Markdown and add front matter such as a title, date, description, authors, tags, and image
- Organize posts by author and tag
- Let readers browse paginated posts and navigate between older and newer entries
- Automatically generate RSS, Atom, or JSON feeds for readers who subscribe
- Control how post summaries are truncated with a truncation marker
- Mark posts as drafts or unlisted so they do not appear in public listings or feeds

## Post listing and pagination

The blog listing shows published posts in reverse chronological order. If you publish more posts than fit on one page, the blog automatically creates additional listing pages.

Each individual post page includes:

- The post content
- Author information
- Tag information
- **Previous** and **Next** navigation to adjacent published posts

Posts marked as drafts or unlisted do not appear in the main listing.

## Authors

You can maintain a shared author list that includes each person’s name, image, URL, and email address. When a post uses an author, that author’s details appear on the post and are included in generated feeds.

This is useful for team blogs where multiple contributors publish under their own names. Author pages can group posts by contributor, making it easier for readers to find everything written by one person.

## Tags

Each post can include one or more tags. Tag pages group related posts together, and tags are also included in generated feeds. This helps readers discover posts on the same topic without needing manual cross-links.

## Feeds

The blog can generate subscription feeds in three formats:

- RSS
- Atom
- JSON

When feeds are enabled, readers can subscribe to your blog with a feed reader. Each feed includes post titles, dates, descriptions, authors, and tags.

You can control several feed settings:

- Which feed formats to generate
- How many posts to include
- The feed title, description, and copyright text
- Whether drafts and unlisted posts are excluded
- Whether to apply custom XML styling with XSLT files

Drafts and unlisted posts are always excluded from generated feeds.

## Blog date and ordering

The date assigned to each post controls its position in the blog listing. Newer posts appear first. Use consistent dates to keep the blog timeline accurate and easy to follow.

## Boundaries

The blog has a few important boundaries to keep in mind:

- Blog navigation is separate from documentation navigation. Blog posts cannot use the documentation sidebar system.
- Feed files are not generated when the website uses hash-based page navigation.
- A post must have a date and be listed to appear in the default public blog listing and feeds.
- The blog listing, tag pages, and author pages follow their own organization and are not part of the document sidebars.

## Multilingual blogs

If your website is available in multiple languages, blog posts can be translated into those languages. The blog preserves the same author, tag, and feed behavior across supported locales.

## Suggested workflow

1. Add a Markdown file for each new post.
2. Set the post title, date, description, and author in the front matter.
3. Add tags so the post appears in relevant tag pages.
4. Use a truncation marker to control the summary shown on listing pages.
5. Publish the post so it appears in the listing and feeds.