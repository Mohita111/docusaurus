# How to initialize a new Docusaurus project

This guide walks you through creating a new Docusaurus website from the official starter template. When you finish, you’ll have a project folder with a working documentation site that you can preview on your computer.

## Before you begin

- Install Node.js on your computer. The creation tool checks your installed Node.js version and stops with a message if it is too old.
- Make sure you have a package manager available. npm is included with Node.js, but you can also use Yarn, pnpm, or Bun if you have one installed.
- Have an internet connection. The tool downloads a template and installs project dependencies.

## Create a new project

1. Open your terminal.
2. Run the following command, replacing `my-website` with the name of the folder you want to create:
   ```bash
   npx create-docusaurus@latest my-website classic
   ```
   The `classic` value tells the tool to use the recommended starter template.

   If you want to choose from all available options interactively, run:
   ```bash
   npx create-docusaurus@latest my-website
   ```
3. If you use the interactive command, answer the prompts:
   - **What should we name this site?** Enter a name for your site. The default is `website`.
   - **Select a template below...** Choose **classic (recommended)** for a complete documentation and blog site.
   - **Which language do you want to use?** Select **JavaScript** or **TypeScript**.
   - **Select a package manager...** Choose the package manager you want to use, such as **npm**.
4. Wait for the creation process to finish. The tool creates the project folder, copies the selected template, and installs dependencies. You’ll see progress messages such as **Creating new Docusaurus project...** and **Installing dependencies with npm...**.
5. When the process is complete, the terminal prints a success message with a list of commands you can run.

## Preview your new site

After the project is created, run the two commands shown in the terminal. For most npm users, the commands are:

```bash
cd my-website
npm start
```

The development server starts and shows the local address, usually `http://localhost:3000`. Open that address in your browser to see the new Docusaurus site.

The starter site includes pages, a documentation area, and a blog, so you can begin adding content right away.

## Common issues and how to fix them

- **“Directory already exists”**  
  A folder with the same name already exists. Choose a different site name or create the project in another location.

- **“Directory not empty”**  
  You used `.` as the site name, but the current folder contains files. Run the command from an empty folder.

- **“Minimum Node.js version not met”**  
  Your Node.js version is too old. Install a newer version of Node.js and try again.

- **No package manager detected**  
  The tool may fall back to npm and show a warning. Install Node.js or your preferred package manager, then retry.

- **“Dependency installation failed”**  
  The project folder was created, but package installation stopped. The terminal shows retry commands. Usually you can run `cd my-website` and then `npm install`.

- **“Invalid template”**  
  The template name you entered is not recognized. Rerun the command without a template argument to see the available choices.

- **Git repository URL rejected**  
  If you choose the **Git repository** template option, enter a URL that begins with `https://` or `git@`.