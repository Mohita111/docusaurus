# How to Publish Blog Posts with Authors and Tags

This guide explains how to create blog posts, assign authors and tags, customize author and tag pages, and turn the blog off entirely when you do not need it.

## Create a Blog Post

Every blog post begins as a dated entry. The date in the name tells your site when the post was published.

You can create a post in one of two ways:

- As a single Markdown file, for example: **2021-08-26-welcome**
- As a folder, for example: **2021-08-26-welcome**, containing a main Markdown file inside it

The folder approach is useful when you want to keep images next to the post. Place the post’s image files inside the same folder and refer to them by their file name in the post body. Your site will load them correctly.

## Add Post Details at the Top

At the top of each post, include a details section that tells your site:

| Field | What it does | Example |
|---|---|---|
| Short URL | Creates a clean, custom web address for the post | `welcome` |
| Title | The display title of the post | `Welcome` |
| Authors | One or more author handles from your Authors list | `slorber`, `yangshun` |
| Tags | One or more tag handles from your Tags list | `facebook`, `hello`, `docusaurus` |

List multiple authors or tags in the same details section, separated by commas.

## Mark the Preview Excerpt

Inside the post body, place a special truncate marker at the point where the preview should end.

Everything before this marker appears in blog listing previews and feeds. Everything after it appears when a reader opens the full post.

If you do not add a truncate marker, your site may show a warning because it cannot tell where the excerpt ends.

## Set Up Author Profiles

Before an author can be added to a post, they must be defined in your Authors list.

For each author, you can set:

- **Short handle** – used when adding the author to a post
- **Display name** – the full name shown on the site
- **Professional title** – shown on the author’s page
- **Website** – a personal or professional site link
- **Profile image** – a URL to the author’s photo
- **Personal author page** – when enabled, your site creates a page listing all articles by that author
- **Custom author page address** – optionally set a custom short web address for that author’s page

### Social Links

Each author can include social links for:

- X
- LinkedIn
- GitHub
- Newsletter

For X, LinkedIn, and GitHub, use the author’s handle. For a newsletter, use the full newsletter signup URL.

### Custom Author Page Address Example

If an author’s standard page would be long or difficult to remember, you can assign a custom address such as **all-sebastien-lorber-articles**. Readers then find all of that author’s posts at that custom location.

## Set Up Tags

Tags help group related posts. Each tag is defined in your Tags list with:

- **Short handle** – used when adding the tag to a post
- **Display label** – the visible tag name
- **Custom tag page address** – a short web address for the tag’s page, such as `facebook` or `docusaurus`
- **Description** – short text shown on the tag page to explain what the tag covers

When you add a tag to a post, your site automatically creates or updates a tag page that lists all posts using that tag.

## Put It All Together

To publish a fully attributed, tagged post:

1. Create the post using a dated name.
2. Add the short URL, title, author handles, and tag handles at the top.
3. Add a truncate marker to define the preview excerpt.
4. Make sure each author handle already exists in the Authors list.
5. Make sure each tag handle already exists in the Tags list.
6. Save the post. Your site automatically places it under the correct date, author page, and tag pages.

## Turn the Blog Off Completely

If you do not want a blog on your site at all:

1. Remove the blog folder from your site.
2. Turn the blog feature off by setting the blog option to **false** in your main site settings.

This removes the blog pages from your site.