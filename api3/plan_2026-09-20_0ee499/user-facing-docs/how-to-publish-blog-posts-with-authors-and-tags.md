# How to Publish Blog Posts with Authors and Tags

This guide explains how to create a blog post, attribute it to one or more authors, assign tags, and turn the blog off if you do not need it.

## Creating a Blog Post

In your blog workspace, create a new entry using a date followed by a short name. The date controls when the post appears.

You can create the post as:

- A single file, for example **2021-08-26-welcome**
- A folder with the same dated name, containing a file called **index**. This is useful when you want to keep images next to the post text.

For a folder-based post, place any images inside the folder and refer to them by their file name, such as `./docusaurus-plushie-banner.jpeg`.

### Post Settings

At the top of the post, include a settings block that defines:

- **slug**: the short link-friendly name, such as `welcome`
- **title**: the post title, such as `Welcome`
- **authors**: one or more author keys, such as `[slorber, yangshun]`
- **tags**: one or more tag keys, such as `[facebook, hello, docusaurus]`

Example settings block:

```
---
slug: welcome
title: Welcome
authors: [slorber, yangshun]
tags: [facebook, hello, docusaurus]
---
```

After the settings block, write your post content.

### Post Preview Marker

Add a truncation marker after the introductory text to control how much of the post appears in the blog list. Use:

```
{/* truncate */}
```

Everything before this marker appears as the preview.

## Adding Authors

The blog uses a central author list, commonly found in the **authors.yml** file. Each author has a unique key that you reference in the `authors` field of a post.

For each author, you can define:

- **name**: the display name
- **title**: the author’s role or short description
- **url**: the author’s main website or profile
- **image_url**: a photo or avatar
- **page**: whether to create a dedicated author page, and optionally a custom link for that page
- **socials**: social media profile links

Example author entry:

```
yangshun:
  name: Yangshun Tay
  title: Ex-Meta Staff Engineer, Co-founder GreatFrontEnd
  url: https://linkedin.com/in/yangshun
  image_url: https://github.com/yangshun.png
  page: true
  socials:
    x: yangshunz
    linkedin: yangshun
    github: yangshun
    newsletter: https://www.greatfrontend.com
```

### Custom Author Page Link

To customize the link for an author’s dedicated page, use the `permalink` option inside the `page` setting:

```
slorber:
  name: Sébastien Lorber
  title: Docusaurus maintainer
  url: https://sebastienlorber.com
  image_url: https://github.com/slorber.png
  page:
    permalink: '/all-sebastien-lorber-articles'
  socials:
    x: sebastienlorber
    linkedin: sebastienlorber
    github: slorber
    newsletter: https://thisweekinreact.com
```

## Defining Tags

Tags are managed in a central tag list, commonly found in the **tags.yml** file. Each tag has a unique key that you reference in the `tags` field of a post.

For each tag, you can define:

- **label**: the display name shown on the site
- **permalink**: the link path for the tag’s archive page
- **description**: a short description of the tag

Example:

```
facebook:
  label: Facebook
  permalink: /facebook
  description: Facebook tag description
```

Use the tag key, such as `facebook`, in your post’s `tags` field to apply it.

## Disabling the Blog

If you do not need a blog, set the blog option to `false` in your site settings. This turns off the blog entirely and removes it from the site.