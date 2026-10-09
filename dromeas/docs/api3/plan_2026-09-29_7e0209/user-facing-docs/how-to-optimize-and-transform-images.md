# How to optimize and transform images

Adding images to your Markdown or MDX content is easy in Docusaurus. When you save your file, Docusaurus automatically transforms the image reference so it loads efficiently, preserves your alt text and titles, and adds the image's dimensions to prevent layout shifts.

## Prerequisites

- A Docusaurus site
- An image file in a supported format, such as PNG, JPEG, GIF, WebP, or SVG
- Permission to edit the Markdown or MDX file where you want to add the image

## Step 1: Place your image file where your page can find it

Choose one of these locations:

- **Same folder as the page:** Put the image next to your Markdown file.
- **Static assets folder:** Place the image in the static folder and reference it with an absolute path, such as `/img/logo.png`.
- **Anywhere in the site folder:** Use the `@site/` alias to point to a path inside your site folder, such as `@site/static/img/logo.png`.

Docusaurus resolves relative paths against the folder of the file that contains the image reference.

## Step 2: Insert the image using Markdown syntax

In your Markdown or MDX file, add an image with the standard Markdown image syntax:

```markdown
![A descriptive alternative text](path/to/image.png)
```

Replace `path/to/image.png` with the actual location of your image.

## Step 3: Add alternative text

The text inside the square brackets is the alternative text. It appears if the image can't be displayed and is used by screen readers.

```markdown
![A diagram of the installation flow](assets/install-flow.png)
```

## Step 4: Optionally add a title

You can add a title by putting text in quotation marks after the image path:

```markdown
![A diagram of the installation flow](assets/install-flow.png "Installation flow")
```

The title appears as a tooltip when a reader hovers over the image.

## Step 5: Let Docusaurus process the image automatically

When you save the file, Docusaurus does the following without any extra input from you:

- It finds the image at the given path.
- It reads the image's width and height and adds those values to the displayed image, so the page layout stays stable while the image loads.
- It bundles the image so it's served efficiently as part of your site.

## Step 6: Skip automatic processing when you need to

If you don't want Docusaurus to process an image reference, prefix the URL with `pathname://`. This is useful when you want the raw URL to pass through unchanged.

```markdown
![External image](pathname://example.com/image.png)
```

## Expected result

After you save and build or preview your site:

- The image appears on the page with your alt text and title (if you provided one).
- The image includes its width and height, preventing visible layout shifts.
- Broken image references produce a warning or a build failure, depending on how your site is configured.

## Common issues

### Missing or misspelled image files

If Docusaurus can't find the image at the referenced path, you'll see a message saying the image couldn't be resolved to an existing local image file. Check the path, capitalization, and file extension, and make sure the file exists in the expected location.

### Empty image URLs

An image syntax with no URL, such as `![]( )`, triggers a warning about an empty image URL. Add a valid path or remove the empty image reference.

### Unreadable or invalid image files

If the image file exists but can't be read as a valid image, Docusaurus warns that the image can't be read correctly. Open the file in an image viewer to confirm it isn't corrupted, and make sure it's in a supported format.

### Controlling broken image behavior

Your site can be configured to either warn about broken images or fail the build. The behavior is controlled by the `markdown.hooks.onBrokenMarkdownImages` setting in your site configuration. Set it to `warn` to continue the build with a warning, or `throw` to stop the build when an image is missing.

### Using the `pathname://` escape hatch

If you intentionally don't want Docusaurus to resolve a URL, prefix it with `pathname://`. Docusaurus removes the prefix and leaves the remaining URL untouched. This avoids broken image checks for that reference.