# System Requirements and Prerequisites

This guide explains everything you need to have in place before you can build and preview either of the two documentation websites in this project. One website is built with a tool called Docusaurus, and the other uses a tool called MkDocs. Both can be set up on the same computer, but they have different requirements.

---

## What You Will Need

### 1. Node.js 20 or Newer (Required for the Main Documentation Site)

The main documentation website requires **Node.js version 20.0 or higher**. This is a runtime that the website uses to build and display pages.

**To check if you have it:**

```bash
node -v
```

If the version number is `20.0` or higher, you are ready.  
If the version is lower, or if the command is not found, you need to install a newer version of Node.js.

---

### 2. Python 3.12 (Required for the Cloud Solutions Documentation Site)

The second documentation site, called **DigitalOcean Solutions Docs**, requires **Python 3.12**. Python is used to build that particular website.

**To check if you have it:**

```bash
python --version
```

or

```bash
python3 --version
```

The version should say `3.12` or a later 3.x release that is compatible. If Python is not installed or is too old, install Python 3.12 from the official Python website.

---

### 3. Package Managers

Each documentation website needs a package manager to install its supporting tools.

#### For the Docusaurus Site (Node.js)

You can use either **Yarn** or **npm**. Both work, but Yarn is the one mentioned in the official setup instructions for this project.

**Check Yarn:**

```bash
yarn --version
```

**Check npm:**

```bash
npm --version
```

If Yarn is missing, you can usually install it with:

```bash
npm install --global yarn
```

#### For the MkDocs Site (Python)

You need **pip**, the package manager for Python. This is usually included with Python 3.12.

**Check pip:**

```bash
pip --version
```

or

```bash
pip3 --version
```

---

### 4. Required Python Packages for the Cloud Solutions Documentation Site

The DigitalOcean Solutions Docs site needs two Python packages:

- `mkdocs` version **1.5.0 or higher**
- `mkdocs-material` version **9.0.0 or higher**

You can install both at once with:

```bash
pip install "mkdocs>=1.5.0" "mkdocs-material>=9.0.0"
```

After installation, you can verify the versions:

```bash
mkdocs --version
```

---

### 5. Git and a GitHub Account (Required Only for Publishing)

To build the sites locally, you do **not** need Git or GitHub. However, if you plan to publish either site to **GitHub Pages**, you must have:

- **Git** installed on your computer
- A **GitHub account**

**Check Git:**

```bash
git --version
```

If Git is not installed, download it from the official Git website.  
For a GitHub account, sign up at [github.com](https://github.com) if you do not already have one.

---

### 6. Recommended Editor Tools for Writing Markdown and MDX

Both documentation sites use Markdown, and the Docusaurus site also uses **MDX**, which is an extended form of Markdown. You do not need a special editing program, but the following editors are helpful:

- **Visual Studio Code** with Markdown and MDX extensions
- **Typora** for a simple, focused Markdown experience
- **Obsidian** for note-style editing with live preview
- Any plain text editor, such as Notepad++, Sublime Text, or Atom

A good editor will highlight text, show page previews, and help you spot formatting errors before you build.

---

## Quick Checklist Before You Start

| Requirement | Needed For | How to Check |
|-------------|------------|--------------|
| Node.js 20+ | Main documentation site | `node -v` |
| Python 3.12 | Cloud solutions documentation site | `python --version` |
| Yarn or npm | Main documentation site setup | `yarn --version` or `npm --version` |
| pip | Cloud solutions documentation site setup | `pip --version` |
| mkdocs 1.5.0+ | Cloud solutions documentation site | `mkdocs --version` |
| mkdocs-material 9.0.0+ | Cloud solutions documentation site | Checked during install |
| Git | Publishing to GitHub Pages | `git --version` |
| GitHub account | Publishing to GitHub Pages | Sign in at github.com |

If every item in this checklist passes, your computer is ready to build both documentation sites locally. If any item is missing, install the missing piece and run the check again.