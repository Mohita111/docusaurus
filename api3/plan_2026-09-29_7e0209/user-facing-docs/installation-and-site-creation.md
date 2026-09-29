# Installation and Site Creation

Create a new Docusaurus site and see it running on your computer in a few minutes.

## Before you begin

Make sure you have Node.js installed. You also need a terminal and a package manager such as npm, Yarn, pnpm, or Bun.

## Step 1: Create a new site

Open your terminal and run:

```bash
npx create-docusaurus@latest
```

This starts an interactive setup.

1. When asked **What should we name this site?**, enter a project name such as `my-website`, then press **Enter**. The default name is `website`.
2. When asked **Select a template below...**, choose **classic (recommended)**.
3. If asked to choose a language, select **TypeScript** or **JavaScript**.
4. If asked to select a package manager, choose the one you want to use.

The setup creates a new folder for your site and installs the required dependencies automatically.

## Step 2: Choose a template

The template decides what your new site includes. The available choices are:

- **classic (recommended)**: A complete site with documentation, a blog, custom pages, and a responsive theme.
- **Git repository**: Start from a public Git repository that already contains a Docusaurus site.
- **Local template**: Start from an existing template folder on your computer.

The classic template is the best starting point for most sites.

You can also skip the interactive template prompt and choose a language directly:

```bash
npx create-docusaurus@latest my-website --typescript
```

Use `--javascript` for a JavaScript project.

## Step 3: Understand your new project

After setup finishes, open your new site folder. You will see these important files and folders:

- `docs/`: Markdown files for your documentation pages.
- `blog/`: Markdown files for blog posts.
- `src/`: React components and custom styling files.
- `static/`: Images and other static files.
- `docusaurus.config.js`: Main site settings such as title, tagline, navigation, and footer.
- `sidebars.js`: Controls the sidebar navigation for your documentation.
- `package.json`: Lists project scripts and dependencies.

The generated site comes with the title **My Site** and the tagline **Dinosaurs are cool**. You can change these in `docusaurus.config.js`.

## Step 4: Preview your site locally

Move into your new project folder and start the development server:

```bash
cd my-website
npm start
```

Replace `my-website` with the name you chose in step 1.

Your terminal shows a local address, usually `http://localhost:3000`. Open that address in your browser to see your new site. The page automatically refreshes when you edit and save a file.

To stop the development server, press **Ctrl+C** in the terminal.

## Useful command options

You can add these options to `npx create-docusaurus@latest` when creating a site:

- `--typescript`: Use TypeScript.
- `--javascript`: Use JavaScript.
- `--package-manager <npm|yarn|pnpm|bun>`: Choose a specific package manager.
- `--skip-install`: Create the project without installing dependencies.
- `--git-strategy <deep|shallow|copy|custom>`: Choose how to copy a Git-based template.

## Next steps

Now that your site is running, you can:

- Open the `docs/` folder to start editing your documentation.
- Change your site title, tagline, colors, and other settings in `docusaurus.config.js`.
- Add new documentation or blog posts by creating Markdown files in `docs/` or `blog/`.

Your new Docusaurus site is ready to personalize.