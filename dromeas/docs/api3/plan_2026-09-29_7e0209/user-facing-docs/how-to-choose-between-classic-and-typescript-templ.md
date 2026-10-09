# How to choose between Classic and TypeScript templates

When you scaffold a new Docusaurus site, the tool asks which template and which language you want to use. This guide walks you through choosing the Classic template and deciding between the JavaScript and TypeScript versions.

## Prerequisites

- You have started the Docusaurus site scaffolding tool in a terminal.
- You have a name for your site, or you are willing to accept the default name **website**.

## Choose the Classic template and your language

1. In the **What should we name this site?** prompt, type a name for your site or press Enter to use **website**.
2. In the **Select a template below...** prompt, choose **classic (recommended)**. The Classic template appears first and is marked as recommended.
3. In the **Which language do you want to use?** prompt, choose one of the following:
   - **TypeScript**: creates the site using a TypeScript version of the Classic template
   - **JavaScript**: creates the site using the standard JavaScript version
4. Wait while the tool creates the site. If prompted, select the package manager you want to use for installing dependencies.

## What the Classic template provides

The Classic template includes a documentation site with a docs section and a blog. It is the recommended starting point for most Docusaurus sites.

## Expected result

- A new site folder is created with the name you entered.
- If you chose **TypeScript**, the site is created with TypeScript files.
- If you chose **JavaScript**, the site is created with JavaScript files.
- The tool confirms that the site was created and shows the commands you can use to work with it.

## Common issues

- If you choose a template that does not offer a TypeScript version, the tool tells you that the selected template does not provide a TypeScript variant and exits. Start the scaffolding process again and choose a different template, or choose **JavaScript**.
- If the site name you entered already exists or the destination folder is not empty, you are asked to provide a different name.
- If you cancel the language prompt, the tool exits without creating a project. Start the scaffolding process again.