# How to optimize and transform images

This guide shows you how to add images to your documentation so they display correctly and are prepared automatically for your site.

## Prerequisites

- You have a documentation page you can edit
- You have an image file saved on your computer or accessible from your site directory

## Add an image to a page

1. Save the image file in a folder that you can reference from your page.
2. Open the page where you want the image to appear in your editor.
3. Insert the image using standard markdown image syntax:

```markdown
   ![A short description of the image](./my-image.png)
   ```

   Replace`./my-image.png` with the actual file name and extension of your image.

4. Add optional hover text after the image path:

```markdown
   ![A short description of the image](./my-image.png "Optional hover text")
   ```

5. Save the page.

## Reference images from the site root

Use the`@site/` shortcut to point to an image stored anywhere in your site project:

```markdown
![Site logo](@site/images/logo.png)
```

This path resolves from the site root, so you can reference images from any page location without worrying about relative paths.

## Reference images from the static directory

Use a path that starts with`/` to point to an image inside your site's static directory:

```markdown
![Profile photo](/images/profile.png)
```

The system looks for`profile.png` inside an`images` folder within the static directory.

## What the system does automatically

When you save your page, Docusaurus automatically processes your images:

- **Detects dimensions**: The system reads your image file and includes its width and height in the HTML output. This prevents layout shift while the image loads.
- **Preserves metadata**: Your alt text and title (hover text) are preserved exactly as you wrote them.
- **Handles the asset**: The image is processed through webpack's asset loader, ensuring it's optimized and served correctly as part of your site build.
- **Generates optimized output**: Image references are converted to require statements internally, so webpack can bundle and optimize them as part of the production build.

## Skip automatic image processing

If you want an image reference to remain unchanged and bypass all automatic processing, use the`pathname://` protocol:

```markdown
![Screenshot](pathname://images/screenshot.png)
```

This is useful when you have an image path that should be served as-is without transformation or dimension detection.

## Common issues

**Image doesn't appear on the page**
- Check that the file path spelling and capitalization match exactly.
- Verify the file extension is correct (`.png`,`.jpg`,`.gif`, and so on).
- Confirm the image file actually exists at the location your path points to.
- Check your browser's developer console for any error messages about missing assets.

**"Markdown image with empty URL" error**
- This happens when you write an image tag with no URL, like`![]()`.
- Always provide a path to an actual image file.

**Wrong path type used**
- Relative paths like`./my-image.png` are resolved from your page's folder. If your image is in a different folder, the path won't work.
- Use`@site/` to reference images from the root of your project structure.
- Use`/` paths only for images in the static directory.

**Image has no dimensions detected**
- If the system can't read your image file to determine its dimensions, the HTML output is generated without width and height attributes.
- This usually happens if the image file is corrupted or in an unsupported format.
- Try converting your image to a common format like PNG or JPEG.

**You want to use a URL path instead**
- If your image path points to a URL like`https://example.com/image.png` rather than a local file, Docusaurus doesn't process it, and it's left as-is in the HTML.
- This is the correct behavior for external images.

## Related features

Image Optimization with LQIP, Learn how to enable Low Quality Image Placeholders for better perceived performance on image-heavy pages.

MDX & Markdown Authoring, Explore the full markdown and MDX authoring capabilities available for your documentation content.
