# How to paginate through blog listing

This guide explains how to move through a blog that spans multiple pages and find posts beyond the first page.

## Prerequisites

- You have opened a Docusaurus site that includes a blog.
- The blog contains more posts than the site owner has set to display on one page. By default, a blog listing shows 10 posts per page.
- You are viewing the blog listing page, not an individual blog post.

## Steps

1. Open the blog by selecting **Blog** in the site navigation.

2. On the blog listing page, browse the post previews shown on the first page.

3. Look for the page navigation controls, usually located below or around the post list. These controls let you move to the next or previous page of posts.

4. To see older posts, select the next page link. For example, select **Next** or the next page number when it's available.

5. To return to more recent posts, select the previous page link or the preceding page number.

6. Continue selecting the next page link until you reach the last page. On the last page, the next page link no longer appears.

7. When you find a post you want to read, select its title or preview to open the full post.

## Expected result

- Each time you move to another page, the blog listing shows a different group of post previews.
- The page navigation updates to reflect your current position.
- On the first page, there's no previous page link; on the last page, there's no next page link.
- The URL changes to reflect the page number (for example,`/blog/page/2`).

## Common issues

- **I don't see page navigation controls.** Your blog likely has 10 or fewer posts, so all posts fit on a single page. Pagination appears only when the number of posts exceeds the configured posts per page.

- **Some posts don't appear in the listing.** Draft or unlisted posts aren't included in the public blog listing. If you expect a post but can't find it on any page, it may be marked as draft or unlisted.

- **Post previews look very long.** Blog listing previews end at a truncation marker. If a post doesn't include one, the listing may show more of the post than expected. This doesn't affect pagination itself.

- **The next page link is missing but I know there are more posts.** Verify that the posts you expect are published and listed, not draft or unlisted.

## Related guides

- How to read blog posts, Learn how to open and read individual blog posts.
- Blog Plugin, Understand how the blog plugin manages post organization and pagination.
