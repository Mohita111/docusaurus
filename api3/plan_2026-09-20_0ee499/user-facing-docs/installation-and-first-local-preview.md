# Installation and First Local Preview

This guide walks you through downloading the project, starting both websites on your computer, and confirming that live editing works.

## What you need before you start

- Git, so you can download the project
- Node.js version 20 or higher, required by the main website
- Yarn, a package manager for the main website
- Python 3 with pip, required by the documentation website

## What you are setting up

The project contains two separate websites:

1. **Main website** — built with Docusaurus. It includes a homepage, a docs area, and a blog.
2. **DigitalOcean Solutions Docs** — built with MkDocs. It is a searchable documentation site focused on DigitalOcean integrations.

Each website runs independently on its own local address.

---

## Step 1: Download the project

1. Copy the repository address from your Git hosting service.
2. Open a terminal on your computer.
3. Clone the repository:

```bash
git clone <repository-address>
```

4. Move into the project folder that was just created.

---

## Step 2: Start the main website

The main website lives in the `my-website` folder.

1. In your terminal, go into that folder:

```bash
cd my-website
```

2. Install the website dependencies:

```bash
yarn
```

3. Start the local development server:

```bash
yarn start
```

This starts the website and usually opens it in your browser automatically. If it does not open, go to:

```text
http://localhost:3000
```

Leave this terminal window running. Most changes you make are reflected live without restarting the server.

---

## Step 3: Start the documentation website

The documentation site lives in the `DigitalOcean-Docs` folder.

1. Open a second terminal window or tab.
2. Go into the documentation folder:

```bash
cd DigitalOcean-Docs
```

3. Install the required Python packages:

```bash
pip install -r requirements.txt
```

4. Start the MkDocs development server:

```bash
mkdocs serve
```

The documentation site becomes available at:

```text
http://127.0.0.1:8000
```

Keep this terminal window running while you preview the site.

---

## Step 4: Confirm both sites look correct

### Main website

- Confirm the homepage displays.
- Open the **Docs** section from the navigation.
- Open the **Blog** section from the navigation.
- Confirm the pages render cleanly and navigation works.

### DigitalOcean Solutions Docs

- Confirm the **Welcome** page displays.
- Use the navigation to open these pages:
  - Bitwarden Deployment
  - Storage with CSI for S3
  - Observability via OpenLIT
  - Database Cluster with Pangolin (CE)
- Confirm the search bar and dark/light mode toggle appear.

---

## Step 5: Make your first small edit and see live reload

### Main website

1. Keep the `yarn start` terminal running.
2. Open the Markdown file for any page you want to change, such as a page inside the Docs or Blog area.
3. Change a visible heading or sentence.
4. Save the file.

The browser updates automatically with your change.

### DigitalOcean Solutions Docs

1. Keep the `mkdocs serve` terminal running.
2. Open the Markdown file for the Welcome page or one of the Core Integrations pages.
3. Change a heading or sentence.
4. Save the file.

The browser refreshes automatically and shows the update.

---

## Troubleshooting

### The main website will not start

- Confirm Node.js version 20 or higher is installed.
- Confirm Yarn is installed.
- Rerun `yarn` inside the `my-website` folder, then run `yarn start` again.

### The documentation site will not start

- Confirm Python 3 and pip are installed.
- Rerun `pip install -r requirements.txt` inside the `DigitalOcean-Docs` folder.
- Run `mkdocs serve` again.

### Changes do not appear in the browser

- Make sure you saved the file.
- Confirm the correct terminal window is still running.
- Check the terminal for error messages.

---

## Expected outcome

You now have both websites running locally:

- The main website is available on your local machine.
- The DigitalOcean Solutions Docs site is available on your local machine.
- You have edited a Markdown file and confirmed that the browser updates automatically.