# Understanding the Repository Layout and Site Workflow

This workspace contains two separate documentation websites, plus a standalone guide at the top level. Each website serves a different audience and has its own publishing process. This guide explains what each area is for, how changes flow from editing to a live site, and where you should make a change for a given task.

---

## What Lives in This Workspace

There are three main areas to know about:

| Area | What it does | Publishing style |
|------|--------------|------------------|
| **Main product website** | The primary site with a homepage, tutorial docs, and a blog | Built and published manually by a maintainer |
| **DigitalOcean deployment guides** | A separate hub for step-by-step infrastructure and deployment instructions | Built and published automatically whenever changes are saved to the main version |
| **Standalone Userguide** | A top-level guide document | Provided as a reference in the workspace |

The two websites are independent. Editing one will not automatically change the other.

---

## The Two Websites at a Glance

### Main Product Website

The main product website is built with a tool called **Docusaurus**. It includes:

- A homepage
- A documentation section with a sidebar
- A blog
- Navigation links and footer links

This is the site to edit when you want to change general product documentation, the homepage, tutorial content, or blog posts.

### DigitalOcean Deployment Guides

The DigitalOcean deployment guides are built with a tool called **MkDocs**. This site is focused on cloud infrastructure and deployment instructions. Its landing page links to guides for:

- Bitwarden deployment
- CSI for S3
- OpenLIT
- Pangolin Community Edition

This is the site to edit when you want to add or update instructions for deploying tools on DigitalOcean.

---

## Main Product Website Workflow

The main product website uses a **manual publishing flow**. That means a person must explicitly run the build and deployment steps.

### 1. Editing content

A contributor edits the relevant content in the main product website area. This could include:

- Updating a documentation page
- Adding or editing a blog post
- Changing the homepage
- Adjusting site navigation or footer links

### 2. Previewing changes locally

Before publishing, the contributor can run a local preview. The site provides a preview command that starts a development server and opens the site in a browser. Most changes appear live as they are made.

### 3. Building the production site

When the content is ready, the contributor runs a build command. This creates a production-ready version of the site in a dedicated output folder.

### 4. Deploying manually

The final step is to run the deployment command manually. This builds the site and pushes the result to the hosting branch used by the live site.

Because this is a manual process, changes to the main product website do not go live until a maintainer decides to deploy them.

---

## DigitalOcean Deployment Guides Workflow

The DigitalOcean deployment guides use an **automatic publishing flow**. No manual build or release command is needed.

### 1. Editing content

A contributor edits the relevant guide page in the DigitalOcean documentation area. The available guide pages are listed on the site's landing page.

### 2. Saving changes to the main version

When the edits are complete, the contributor saves them to the main version of the workspace. In practice, this usually means merging or pushing changes to the default branch.

### 3. Automatic build and deployment

As soon as changes are saved to the main version, an automation service called **GitHub Actions** takes over. The automation:

1. Checks out the latest content
2. Installs the required site-building tools
3. Builds the documentation site
4. Uploads the built site
5. Publishes it to GitHub Pages

No one has to run a local build or deploy command for this site. The entire process happens automatically after changes reach the main version.

---

## Which Site to Edit for Which Content

Use this table to decide where to make a change:

| I want to... | Edit this area |
|--------------|----------------|
| Update general product documentation | Main product website |
| Add or edit a tutorial | Main product website |
| Publish a new blog post | Main product website |
| Change the product homepage | Main product website |
| Add a new DigitalOcean deployment guide | DigitalOcean deployment guides |
| Update Bitwarden, CSI for S3, OpenLIT, or Pangolin instructions | DigitalOcean deployment guides |
| Reference overall project guidance | Standalone Userguide |

If a change is about **how to deploy something on DigitalOcean**, it belongs in the DigitalOcean deployment guides. If a change is about **the product itself, its tutorials, or blog content**, it belongs in the main product website.

---

## Relationships Between Homepage, Docs Sidebar, and Blog

The main product website connects its main sections through the top navigation and footer.

### Top navigation

The top navigation bar includes:

- **Tutorial**: opens the documentation area with the tutorial sidebar
- **Blog**: opens the blog section
- **GitHub**: a link to the associated code repository

### Documentation sidebar

The documentation section has a sidebar that lists tutorial pages. Users can browse the sidebar to move between documentation pages.

### Footer links

The footer also links to:

- The tutorial documentation
- The blog
- Community resources
- The associated code repository

Because the homepage, docs, and blog are all part of the same main product website, navigation among them is built into the site.

The DigitalOcean deployment guides are separate. They have their own landing page and their own guide pages, and they are not linked into the main product website's navigation.

---

## End-to-End Publishing Paths

### Main product website

```
Edit content → Preview locally → Build the site → Run manual deployment → Live site updates
```

This path requires a maintainer to complete the final build and deployment steps.

### DigitalOcean deployment guides

```
Edit content → Save to main version → GitHub Actions builds and publishes automatically → Live site updates
```

This path requires no manual deployment step after the change is saved.

---

## Summary

- The workspace contains two independent documentation sites and a standalone guide.
- The main product website is for general product docs, tutorials, and blog posts. It is published manually.
- The DigitalOcean deployment guides are for infrastructure and deployment instructions. They are published automatically through GitHub Actions.
- To decide where to make a change, ask whether the content is general product information or DigitalOcean-specific deployment guidance.
- The main product website links its homepage, docs, and blog together through navigation and footer links. The DigitalOcean guides are separate from that navigation.