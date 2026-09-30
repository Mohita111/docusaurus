# How to generate project structure with default content

You can create a new Docusaurus website that comes with a ready-to-use starter layout, documentation section, blog, and theme styling. The generator copies a recommended template into a new folder and installs dependencies so you can start writing content right away.

## Prerequisites

Before you begin, make sure you have:

- A supported version of Node.js installed
- One of these package managers available: `npm`, `yarn`, `pnpm`, or `bun`
- Access to a terminal or command prompt
- An empty location where you want the new website folder to be created

The generator will stop if it detects an existing folder or a non-empty folder.

## Generate a new project with default content

1. Open your terminal and navigate to the folder where you want the new website folder to appear.

2. Run the project generator:

   ```bash
   npx create-docusaurus@latest
   ```

   You can also name the site and choose the recommended template directly:

   ```bash
   npx create-docusaurus@latest my-website classic
   ```

3. When prompted with **What should we name this site?**, enter a name or press **Enter** to use the default name `website`.

4. When prompted with **Select a template below...**, choose **classic (recommended)**.

5. When prompted with **Which language do you want to use?**, choose **JavaScript** or **TypeScript**.

6. If multiple package managers are installed, choose one from the **Select a package manager...** prompt. If only one is installed, the generator selects it automatically.

7. Wait while the generator copies the default content and installs dependencies.

## Expected result

A new folder is created with the name you provided. The folder contains a complete starter website, including:

- A home page
- A documentation section
- A blog section
- Default theme styling and navigation

The generator prints a success message similar to this:

```text
Created my-website.
Inside that directory, you can run several commands:

  npm start
    Starts the development server.

  npm run build
    Bundles your website into static files for production.
```

To see your new site, run the development server:

```bash
cd my-website
npm start
```

The development server starts, and you can view the default website in your browser.

## Common issues

### The destination folder is not empty or already exists

The generator verifies the target location before copying files. You might see:

- **Directory not empty at ...**
- **Directory already exists at ...**

To resolve this, choose a new website name or remove the existing folder before running the generator again.

### No package manager is available

If the generator cannot detect an installed package manager, it warns:

```text
No package maintainer available? Trying with npm
```

It then falls back to `npm`. If `npm` is not installed, dependency installation will fail.

### Dependency installation fails

If dependencies cannot be installed, the generator displays:

```text
Dependency installation failed.
```

The website folder is still created. You can retry installation manually:

```bash
cd my-website
npm install
```

Replace `npm` with your chosen package manager if you used a different one.

### Invalid package manager choice

If you use the `--package-manager` option, the value must be one of:

- `npm`
- `yarn`
- `pnpm`
- `bun`

Any other value causes the generator to stop with an invalid package manager message.