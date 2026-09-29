# Install Docusaurus and create your first site

Docusaurus helps you create documentation websites quickly. This guide walks you through creating a new project, choosing a starting template, and running the site on your computer.

## Before you begin

- Install Node.js. The installer includes npm, the package manager you'll use to run commands.
- Open a terminal in the folder where you want to create the site.

## 1. Create a new site

Run this command to create a project named `my-website` with the recommended **classic** template:

```bash
npx create-docusaurus@latest my-website classic
```

You can replace `my-website` with any name you want. The name must not contain spaces and must not already be an existing folder.

If you prefer to answer prompts instead, run:

```bash
npx create-docusaurus@latest
```

The tool asks:

- **What should we name this site?** The default suggestion is `website`.
- **Select a template below...** Choose a starting layout.

### Available template options

The menu lists built-in templates, with **classic (recommended)** shown first. The classic template includes:

- A documentation section
- A blog section
- A navigation bar and footer
- A sidebar for documentation navigation

You can also choose:

- **Git repository** to use a template from a public Git repository
- **Local template** to use a folder on your computer

### Choose a language

After picking a template, the tool asks whether you want **TypeScript** or **JavaScript**. The classic template supports both. If a template doesn't offer a TypeScript variant, the tool asks you to choose JavaScript instead.

### Automatic dependency installation

After the files are created, the tool detects your package manager and installs the project dependencies for you. If you have more than one package manager available, it asks you to choose one.

You can skip automatic installation by adding `--skip-install` to the create command. Then you can run the install command yourself later.

## 2. Start the development server

Move into the new project folder:

```bash
cd my-website
```

Then start the local development server:

```bash
npm start
```

If you chose a different package manager during creation, use the matching command, such as `yarn start` or `pnpm start`.

The server starts and shows a local address in the terminal, usually `http://localhost:3000`. Open that address in your web browser to see your new site.

The development server automatically reloads when you change content.

## 3. Understand the generated structure

After the create command finishes, your project folder includes everything you need to start writing.

### Documentation and blog folders

- `docs` — Add Markdown files here to create documentation pages. The sidebar automatically lists them by default.
- `blog` — Add Markdown files here to create blog posts.

### Customization folders

- `src` — Add custom pages and styles. The create tool generates a `src/css/custom.css` file you can edit to change colors and other styles.
- `static` — Store files that should be served exactly as they are, such as images. The template includes an `img` folder for the site logo, favicon, and social card.

### Configuration files

- `docusaurus.config.js` — The main site settings file. Change the site title, tagline, URL, navigation bar, footer, and other preferences here.
- `sidebars.js` — Controls the documentation sidebar. By default, the sidebar is generated from the contents of the `docs` folder.
- `package.json` — Stores the project name, scripts, and dependencies. The `start` script runs the development server.

## Common command options

You can include these options with the create command to skip some prompts:

- `--typescript` — Create the TypeScript version of the chosen template, if available
- `--javascript` — Create the JavaScript version
- `--skip-install` — Skip automatic dependency installation
- `--package-manager <manager>` — Use a specific package manager
- `--git-strategy <strategy>` — Choose how to clone a Git repository template: `deep`, `shallow`, `copy`, or `custom`

Example:

```bash
npx create-docusaurus@latest my-website classic --typescript
```

## What to do next

- Open the `docs` folder and start writing your first page.
- Edit the navigation bar and footer in the site settings file.
- Add a logo and other images to the `static/img` folder.
- When you're ready to share your site, build a production version with your package manager's build command.