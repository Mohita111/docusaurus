# How to localize documentation paths

When you run your Docusaurus site in multiple languages, your documentation paths automatically adjust to reflect each language. This guide shows you how to configure localized documentation paths so visitors see content in their preferred language.

## What you'll accomplish

You'll set up your documentation so that each language version has its own URL structure:

- English:`/docs/guides/getting-started`
- French:`/fr/docs/guides/getting-started`
- Spanish:`/es/docs/guides/getting-started`

The documentation plugin handles this localization automatically once you configure the base paths and languages.

## Prerequisites

- A Docusaurus site with the documentation plugin already configured in`docusaurus.config.js`
- Multiple languages configured on your site
- Your documentation content organized in the content directory (usually`docs/`)

## How localized documentation paths work

The documentation plugin uses two main settings to create localized paths:

**`routeBasePath`**, The base URL where documentation appears (for example,`/docs`).

**`editLocalizedFiles`**, Whether to target the translated version of files when generating edit links, or always target the original language files.

Docusaurus automatically prepends the locale code to paths based on your`i18n.locales` configuration in`docusaurus.config.js`.

## Step-by-step configuration

### 1. Open your site configuration file

Navigate to and open`docusaurus.config.js` at the root of your project.

### 2. Set the base route for documentation

Inside the`presets` section, locate the documentation plugin configuration. Set the`routeBasePath` option to control where your docs appear:

```javascript
presets: [
  [
    '@docusaurus/preset-classic',
    {
      docs: {
        routeBasePath: 'docs', // Docs appear at /docs or /fr/docs, /es/docs, etc.
        path: 'docs',          // Source content directory
        sidebarPath: require.resolve('./sidebars.js'),
      },
    },
  ],
],
```

### 3. Configure localized edit links (optional)

If you want edit links to point to translated files when they exist, set`editLocalizedFiles` to`true`:

```javascript
docs: {
  routeBasePath: 'docs',
  editLocalizedFiles: true,      // Edit links target translated files
  editUrl: 'https://github.com/myorg/myrepo/tree/main',
  editCurrentVersion: true,
}
```

When`editLocalizedFiles` is`true`, the edit URL is computed as:
- **For French**:`editUrl + '/i18n/fr/docusaurus-plugin-content-docs/current/' + docPath`
- **For English**:`editUrl + '/docs/' + docPath`

When`false` (the default), all edit links point to the original language files regardless of which language the visitor is viewing.

### 4. Verify your locale configuration

In the`i18n` section of`docusaurus.config.js`, confirm your language locales are set:

```javascript
i18n: {
  defaultLocale: 'en',
  locales: ['en', 'fr', 'es'],
  localeConfigs: {
    en: {label: 'English'},
    fr: {label: 'Français'},
    es: {label: 'Español'},
  },
}
```

### 5. Build and test your site

Run the build command to generate your site with localized paths:

```bash
npm run build
```

Visit your site and navigate using the language selector to verify paths include locale codes:

- Home page in English:`/docs/guides/getting-started`
- Same page in French:`/fr/docs/guides/getting-started`

## Expected result

After configuration, your documentation becomes accessible at locale-specific URLs. The sidebar, navigation, and all internal links automatically adjust to reflect the current language. Edit buttons (if enabled) link to the correct language version of the source files.

## Troubleshooting common issues

**Paths don't show locale codes**

Verify that`i18n.locales` contains the languages you want. The default locale doesn't receive a prefix in URLs, so`/docs/page` is correct for the default language—add the locale manually only for non-default languages.

**Edit links point to wrong files**

Check that`editLocalizedFiles` matches your intent. If set to`true`, your translated content must be in the expected directory:`i18n/{locale}/docusaurus-plugin-content-docs/current/`. If files aren't there, edit links will be invalid.

**Documentation not appearing for a language**

Ensure you've translated your documentation files and placed them in the correct locale directory. The plugin only generates pages for content that exists. Run`docusaurus write-translations` to extract strings and set up your localization structure.

**Sidebar doesn't reflect new locale**

If you're using versioned documentation, check that your sidebar configuration (`sidebarPath`) includes all necessary sidebar files for each language. Versioned sidebars are placed in`versioned_sidebars/` and locale-specific versions in`i18n/{locale}/docusaurus-plugin-content-docs/`.

## Related

Documentation Plugin, Learn how the documentation plugin organizes and structures your content.

Translation Extraction, Understand how to extract and manage translatable strings.

Configure documentation sidebar and paths, Set up your sidebar and adjust documentation structure.
