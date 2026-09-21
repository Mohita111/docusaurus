# How to Build and Deploy the Docusaurus Site

This guide walks you through turning your Docusaurus project into a live website that anyone can visit. You will learn how to build the site, preview the finished result, and publish it to GitHub Pages.

## Before You Begin

Make sure you have:

- Node.js version 20 or newer installed
- Yarn installed
- An internet connection
- A GitHub account

Start by installing all required project dependencies. Run this command from the project folder:

```bash
yarn
```

This only needs to be done once, or after dependencies change.

---

## 1. Build the Site

The build step creates a final set of static website files that can be hosted anywhere. These files are placed into a folder called `build`.

To create the build:

```bash
yarn build
```

What happens:

- Docusaurus reads all your pages, blog posts, and documentation.
- It generates optimized HTML, CSS, JavaScript, and image files.
- The finished files are written to the `build` folder.

You can take the contents of the `build` folder and upload them to any static hosting service.

---

## 2. Preview the Production Build

Before publishing, it is a good idea to see exactly what visitors will see.

After running `yarn build`, preview the built site locally with:

```bash
yarn serve
```

This starts a local server that serves the contents of the `build` folder. Open the address shown in the terminal to view the production version of your site.

This is different from the regular development preview. The production preview uses the same optimized files that will be deployed.

---

## 3. Deploy to GitHub Pages

Docusaurus can publish your site directly to GitHub Pages. The deployment command builds the site and pushes it to a special branch called `gh-pages`.

There are two ways to deploy, depending on how you connect to GitHub.

### Option A: Deploy Using SSH

If you use SSH keys to connect to GitHub, run:

```bash
USE_SSH=true yarn deploy
```

This tells Docusaurus to use your SSH connection.

### Option B: Deploy Using HTTPS

If you use HTTPS with your GitHub username, run:

```bash
GIT_USER=YourGitHubUsername yarn deploy
```

Replace `YourGitHubUsername` with your actual GitHub username.

For example, if your GitHub username is `alex123`, run:

```bash
GIT_USER=alex123 yarn deploy
```

You may be prompted for a password or personal access token during the process.

### What Deployment Does

The deployment command:

1. Builds your site.
2. Creates or updates the `gh-pages` branch in your GitHub repository.
3. Pushes the built site files to that branch.

Your site then becomes available through GitHub Pages.

---

## 4. Set Your GitHub Pages Target

For deployment to work correctly, two settings in your site configuration must match your GitHub account and repository:

- **Organization name** — usually your GitHub username or organization name
- **Project name** — usually the name of your GitHub repository

For example, if your GitHub username is `facebook` and your repository is named `docusaurus`, the configuration should use:

- Organization name: `facebook`
- Project name: `docusaurus`

These settings tell Docusaurus exactly where to push the `gh-pages` branch.

You may also need to set the site base URL. For GitHub Pages project sites, this is often set to `/<projectName>/`. For example:

```
/MyRepoName/
```

If you are using a custom domain or a personal GitHub Pages site, this may be set to `/` instead.

Check these settings before deploying if your site does not appear where expected.

---

## 5. Clear Caches When Builds Misbehave

Sometimes builds fail or produce unexpected results because of stale cached data. Docusaurus provides a command to clear these caches:

```bash
yarn clear
```

This removes temporary build data and can resolve many build problems.

After clearing, try building again:

```bash
yarn build
```

If the issue was caused by stale cache data, the build should now work correctly.

---

## 6. End-to-End Publishing Workflow

Follow these steps in order to publish your site:

1. **Install dependencies**

   ```bash
   yarn
   ```

2. **Review your site configuration**

   Make sure your organization name and project name match your GitHub repository.

3. **Build the site locally**

   ```bash
   yarn build
   ```

4. **Preview the production build**

   ```bash
   yarn serve
   ```

   Review the site carefully.

5. **Deploy to GitHub Pages**

   Using SSH:

   ```bash
   USE_SSH=true yarn deploy
   ```

   Or using HTTPS:

   ```bash
   GIT_USER=YourGitHubUsername yarn deploy
   ```

6. **Visit your live site**

   Open your GitHub Pages URL in a browser. Your site is now published.

---

## Summary

| Task | Command |
| --- | --- |
| Install dependencies | `yarn` |
| Build the site | `yarn build` |
| Preview production build | `yarn serve` |
| Deploy with SSH | `USE_SSH=true yarn deploy` |
| Deploy with HTTPS | `GIT_USER=YourGitHubUsername yarn deploy` |
| Clear caches | `yarn clear` |

Always preview your build before publishing. If something looks wrong, clear caches and rebuild.