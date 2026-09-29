# System Requirements and Prerequisites

Docusaurus builds static websites, so the setup requirements are intentionally small. You need a current version of Node.js and one package manager. You don't need a database, a backend server, or separate hosting software to start.

## Minimum software requirements

- **Node.js**: version 24.21 or newer
- **Package manager**: npm, Yarn, or pnpm
- **Database or backend**: none required

### Node.js

Docusaurus requires Node.js version 24.21 or later. Older versions won't work reliably. If you're not sure which version you have, you can check it in a terminal:

```bash
node --version
```

The output should show version 24.21 or later, for example `v24.21.0`.

### Package manager

You can use npm, Yarn, or pnpm to install dependencies and run Docusaurus commands.

- npm is included with Node.js, so you likely already have it installed.
- If you use pnpm, use version 12.3.4 or newer.
- If you use Yarn, use a recent stable version.

You can verify your package manager with one of these commands:

```bash
npm --version
pnpm --version
yarn --version
```

### No database or backend required

Docusaurus generates static HTML, CSS, and JavaScript files from your content. Because the output is static, you don't need to configure a database, run a backend service, or set up server-side software before you begin.

## Verify your environment

Follow these steps to confirm your computer is ready for Docusaurus.

1. Open a terminal on your computer.
2. Check your Node.js version:

   ```bash
   node --version
   ```

   You need version 24.21 or later.
3. Check your package manager version:

   ```bash
   npm --version
   ```

   Or, if you use pnpm or Yarn:

   ```bash
   pnpm --version
   yarm --version
   ```

   If you use pnpm, make sure the version is 12.3.4 or later.
4. If a command isn't recognized, install the missing tool:
   - Install Node.js and npm from the Node.js website.
   - Install pnpm from the pnpm website.
   - Install Yarn from the Yarn website.
5. Run the version commands again. When Node.js is 24.21 or later and your package manager is available, your environment is ready.

## Optional requirements

### Search

If you want full-site search later, you can create a free Algolia account. This is optional and isn't required to create, preview, or build a Docusaurus site.

## Next steps

After your environment is ready, you can continue to the installation guide and create your first Docusaurus site.