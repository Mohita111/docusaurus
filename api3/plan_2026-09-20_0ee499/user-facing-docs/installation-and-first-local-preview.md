# Installation and First Local Preview

This guide walks through getting both documentation sites running on your computer, checking that each part renders, and confirming live editing works.

## Before you start

You need:

- **Node.js 20 or newer** for the Docusaurus site
- **Yarn** for installing Docusaurus dependencies
- **Python with pip** for the DigitalOcean docs site
- A terminal application

## 1. Get the project onto your computer

Open a terminal and copy the project from GitHub:

```bash
git clone https://github.com/Mohita111/API3.git
```

Move into the project folder:

```bash
cd API3
```

The project contains two independent documentation sites:

- `my-website` — a Docusaurus site
- `DigitalOcean-Docs` — an MkDocs Material site

Each site has its own install command and start command. It helps to run them in two separate terminal tabs.

## 2. Start the Docusaurus site

In the first terminal tab, move into the Docusaurus site folder:

```bash
cd my-website
```

Install the dependencies with Yarn:

```bash
yarn
```

Start the local development server:

```bash
yarn start
```

This starts a live-reload preview and usually opens your browser automatically. If it does not open, go to:

```text
http://localhost:3000
```

Keep this terminal running while you view the site.

## 3. Start the DigitalOcean docs site

Open a second terminal tab and move into the DigitalOcean docs folder:

```bash
cd DigitalOcean-Docs
```

Install the MkDocs dependencies with pip:

```bash
pip install -r requirements.txt
```

Start the MkDocs live server:

```bash
mkdocs serve
```

MkDocs typically runs at:

```text
http://127.0.0.1:8000
```

Keep this terminal running as well.

## 4. Verify the pages render correctly

### Docusaurus site

In the Docusaurus browser window, check:

- The homepage loads.
- The Docs section is present and opens.
- The Blog section is present and opens.

### DigitalOcean docs site

In the MkDocs browser window, check:

- The Welcome page loads.
- The Core Integrations navigation area appears.
- These navigation entries are visible:
  - Bitwarden Deployment
  - Storage with CSI for S3
  - Observability via OpenLIT
  - Database Cluster with Pangolin (CE)

Click through each entry to confirm the pages open.

## 5. Make a small edit and watch it hot-reload

### Docusaurus live edit

1. Keep `yarn start` running.
2. Open any Markdown content page from the Docusaurus site in a text editor.
3. Change a heading or a sentence.
4. Save the file.

The browser preview updates automatically without restarting the server.

### DigitalOcean docs live edit

1. Keep `mkdocs serve` running.
2. Open a Markdown page from the navigation, such as the Bitwarden Deployment page.
3. Change a sentence or heading.
4. Save the file.

The MkDocs browser preview refreshes automatically.

## Expected outcome

You now have:

- The Docusaurus site running on `localhost:3000`
- The DigitalOcean docs site running on `127.0.0.1:8000`
- Confirmation that pages and navigation render
- Confirmation that editing a Markdown file updates both sites live in the browser

To stop either server, click into its terminal and press `Ctrl + C`.