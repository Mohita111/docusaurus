# How to Configure the Docusaurus Site

This guide explains how to adjust the main configuration file so your Docusaurus site reflects your own organization, brand, hosting location, and navigation structure. The steps below cover the key areas you will likely want to change.

---

## Before You Begin

Open the main configuration file in a plain text editor. This single file controls the most important site-wide settings, including the site name, hosting address, navigation bar, footer, and link-checking behavior.

After making changes, save the file and rebuild or restart your site preview to see the results.

---

## 1. Site Identity

These settings define how visitors recognize your site.

| Setting | Current Value | What to Do |
|---|---|---|
| Site title | `My Site` | Replace with your product or organization name. This appears in the browser tab and at the top of the site. |
| Tagline | `Dinosaurs are cool` | Replace with a short, memorable description of your site. This appears on the home page below the title. |
| Favicon | `img/favicon.ico` | Replace the favicon image with your own. The file name can stay the same, or update the setting if you use a different name. |
| Production URL | `https://your-docusaurus-site.example.com` | Replace with the final web address where your site will be hosted, such as `https://docs.yourcompany.com`. |
| Base path | `/` | Keep as `/` if your site is hosted at the root of a domain. If your site will live inside a subfolder or a GitHub Pages project, change this to match that path, such as `/project-name/`. |

---

## 2. GitHub Pages Settings

If you plan to publish your site with GitHub Pages, update these two values.

| Setting | Current Value | What to Do |
|---|---|---|
| Organization name | `facebook` | Replace with your GitHub username or organization name. |
| Project name | `docusaurus` | Replace with the name of your GitHub repository. |

If you are not using GitHub Pages, you can leave these values as they are, but it is still a good idea to replace the placeholder information so the configuration accurately reflects your project.

---

## 3. Broken-Link Handling and Future Compatibility

### Broken Links

The site is currently set to stop the build process whenever it finds a broken link. This is a strict setting that helps keep your published site free of dead links. If a link points to a page that does not exist, the build fails and you are prompted to fix the link before publishing.

This behavior is useful for maintaining a healthy site. If you later need a less strict approach, your developer can adjust the setting to warn instead of stopping the build.

### Future Compatibility

A compatibility flag called `v4` is currently enabled. This prepares the site for upcoming Docusaurus version 4 behavior. Keep this setting enabled unless you have a specific reason to change it.

---

## 4. Content Areas and Visual Theme

The site uses the standard Docusaurus setup with three main areas:

- **Documentation**
- **Blog**
- **Visual theme**

### Documentation

The documentation area includes an “Edit this page” link on each page. This link currently points to a placeholder repository owned by Facebook/Docusaurus. Change it to point to your own repository so that contributors are taken to the correct source files.

### Blog

The blog section has the following features enabled:

- Reading time is shown on each blog post.
- RSS and Atom feeds are generated so readers can subscribe.
- Blogging best-practice warnings are active for inline tags, inline authors, and untruncated posts.

The blog also has an “Edit this page” link that points to the same placeholder repository. Update it to your own repository.

### Visual Theme

The visual theme uses a custom styling file where you can add your own brand colors and other visual adjustments. This guide does not cover advanced styling, but the theme is already wired up and ready for customization.

### Code Block Colors

Code blocks in your content use two color themes:

- A light theme for bright mode
- A dark theme for dark mode

No change is required unless you want different code block styling.

---

## 5. Navigation Bar

The navigation bar appears at the top of every page. You can customize its title, logo, and menu items.

### Title and Logo

| Setting | Current Value | What to Do |
|---|---|---|
| Navbar title | `My Site` | Replace with the same site title you used in the Site Identity section. |
| Logo image | `img/logo.svg` | Replace the logo image with your own. The file name can stay the same, or update the setting if you use a different name. |
| Logo alt text | `My Site Logo` | Update to describe your logo for accessibility. |

### Menu Items

The current navigation menu includes three items:

1. **Tutorial** — links to the documentation section from the left side of the menu.
2. **Blog** — links to the blog page from the left side of the menu.
3. **GitHub** — links to an external GitHub repository from the right side of the menu. This currently points to a Facebook/Docusaurus repository and should be updated to your own repository.

You can keep these items, change their labels, update the target links, or add new items for pages such as “About,” “Contact,” or “Community.”

---

## 6. Footer

The footer appears at the bottom of every page. It uses a dark background and includes three columns of links.

### First Column: Docs

- Contains a link labeled **Tutorial** that goes to the documentation section.

You can keep this link or change it to point to a different starting page.

### Second Column: Community

This column currently contains:

- **Stack Overflow**
- **Discord**
- **X** (formerly Twitter)

Update these links to point to your own community channels, or replace them with links to your preferred platforms such as Slack, LinkedIn, or YouTube.

### Third Column: More

This column currently contains:

- **Blog** — links to the blog page.
- **GitHub** — links to a placeholder GitHub repository that should be updated to your own.

### Copyright

The footer copyright currently reads:

`Copyright © [current year] My Project, Inc. Built with Docusaurus.`

Replace `My Project, Inc.` with your organization or company name. The year updates automatically based on the current calendar year.

---

## 7. Rebranding Checklist

Use this checklist to make sure you have updated every important setting:

- [ ] Site title
- [ ] Tagline
- [ ] Favicon
- [ ] Production URL
- [ ] Base path
- [ ] GitHub organization name
- [ ] GitHub project name
- [ ] “Edit this page” links in documentation and blog
- [ ] GitHub link in the navigation bar
- [ ] Footer community links
- [ ] Footer GitHub link
- [ ] Copyright text

After completing the checklist, save the configuration, rebuild the site, and verify that the new branding, navigation, and footer appear correctly.