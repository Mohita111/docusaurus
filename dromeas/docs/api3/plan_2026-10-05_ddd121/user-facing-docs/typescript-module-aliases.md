# TypeScript module aliases

TypeScript Module Aliases provides comprehensive type definitions for Docusaurus core modules, giving you full type safety and IntelliSense support when building custom themes and plugins. Instead of working with untyped imports, you get accurate TypeScript types for every Docusaurus module you use.

## Who this is for

This feature is designed for **theme and plugin developers** who want to build type-safe extensions for Docusaurus. If you're creating custom themes, designing plugins, or extending Docusaurus functionality with React components, TypeScript Module Aliases ensures your code has proper type checking and better IDE support.

## What it enables

TypeScript Module Aliases exposes typed imports for the most commonly used Docusaurus modules, organized into several categories:

### Generated modules

When Docusaurus builds your site, it generates several modules that contain runtime data about your configuration and structure. These aliases provide types for:

- **`@generated/docusaurus.config`**, Your full site configuration as a typed object, letting you safely access settings like site title, URL, and theme options in your code
- **`@generated/site-metadata`**, Metadata about your site including language, version, and other computed properties
- **`@generated/site-storage`**, Browser storage utilities for persisting user preferences and state
- **`@generated/registry`**, The registry of all plugins and their configurations
- **`@generated/routes`**, The complete set of routes your site exposes, fully typed as React Router configuration objects
- **`@generated/routesChunkNames`**, Maps routes to their corresponding code-split chunks
- **`@generated/globalData`**, Plugin-provided global data accessible throughout your site
- **`@generated/i18n`**, Internationalization settings and locale information
- **`@generated/codeTranslations`**, Translated strings available in your codebase
- **`@generated/client-modules`**, Client-side modules and lifecycle hooks registered by plugins

### Theme component aliases

When you override theme components or create a custom theme, you can safely import and extend the original implementations:

- **`@theme-original/*`**, Access the original, unoverridden theme component for any path
- **`@theme-init/*`**, Access the initial theme implementation before any swizzling

### Core docusaurus components

Common components you'll use when building themes and plugins come with full type support:

- **`@docusaurus/Head`**, Add elements to the document head (meta tags, scripts, stylesheets) using a Helmet-based API
- **`@docusaurus/Link`**, Render links with automatic base URL handling and broken link detection
- **`@docusaurus/Layout`**, The main layout wrapper for pages
- **`@docusaurus/Error`**, Display error boundaries with fallback UI
- **`@docusaurus/Root`**, The top-level component that wraps your entire site
- **`@docusaurus/ThemeProvider`**, Provide theme context to child components
- **`@docusaurus/BrowserOnly`**, Render components only in the browser, not during static generation
- **`@docusaurus/Noop`**, A no-op component for conditional rendering

### Hooks for accessing site data

React hooks let you read configuration and state from within your components:

- **`useDocusaurusContext()`**, Access the full Docusaurus context including config, siteMetadata, and version info
- **`useRouteContext()`**, Get information about the current route and plugin context
- **`useGlobalData()`**, Read data provided by any plugin across your site
- **`useBaseUrl()`**, Transform relative URLs into absolute URLs with your site's base path
- **`useIsBrowser()`**, Check whether your code is running in a browser or during static generation
- **`useBrokenLinks()`**, Collect links and anchors for validation during development
- **`useIsomorphicLayoutEffect()`**, Use layout effects that work safely during server-side rendering

### Utility modules

Helper modules for common tasks:

- **`@docusaurus/Translate`**, Translate and interpolate strings with placeholders, supporting both function and component syntax
- **`@docusaurus/Interpolate`**, Insert dynamic values into strings with type-safe placeholder detection
- **`@docusaurus/router`**, Export React Router utilities like`useHistory`,`useLocation`,`Redirect`, and`matchPath`
- **`@docusaurus/ExecutionEnvironment`**, Check capabilities of the current environment (DOM support, event listeners, IntersectionObserver, viewport)
- **`@docusaurus/ComponentCreator`**, Create lazy-loaded components using React Loadable for code splitting
- **`@docusaurus/isInternalUrl`**, Detect whether a URL is internal to your site or external
- **`@docusaurus/renderRoutes`**, Render React Router route configurations into React elements
- **`@docusaurus/constants`**, Access Docusaurus constants like`DEFAULT_PLUGIN_ID`
- **`@docusaurus/ErrorBoundary`**, Wrap components to catch and handle React errors gracefully

### Asset imports

TypeScript aliases ensure you can safely import and use various asset types:

- **`.svg` files**, Import as React components with TypeScript types for SVG props
- **`.module.css` files**, Import with typed class name maps
- **`.css` files**, Import stylesheet URLs
- **`.md` and`.mdx` files**, Import as React components

## Key benefits

**Type safety**, Catch errors before runtime. Your IDE knows what props each component accepts, what each hook returns, and which methods exist on generated modules.

**Better IntelliSense**, Get autocomplete suggestions for all Docusaurus APIs. No more guessing at property names or hunting through documentation.

**Easier refactoring**, Change Docusaurus versions or restructure your config with confidence. TypeScript tells you exactly what broke.

**Self-documenting code**, Types serve as inline documentation. You can see what data is available and how to use it without leaving your editor.

**Plugin and theme confidence**, Build extensions knowing they'll work correctly across different Docusaurus setups and versions.

## How to use it

No setup is required. When you create a Docusaurus project with TypeScript support enabled, these type aliases are automatically available. Import from the aliases just as you would from any npm package:

```typescript
import config from '@generated/docusaurus.config';
import {useDocusaurusContext} from '@docusaurus/useDocusaurusContext';
import {Translate} from '@docusaurus/Translate';
import Layout from '@theme/Layout';
```

Your TypeScript compiler and IDE will immediately provide types and autocomplete for all these imports.

## Options and configuration

TypeScript Module Aliases work automatically when you have a`tsconfig.json` in your project. The type definitions are included in the`@docusaurus/module-type-aliases` package, which is installed as a dependency of Docusaurus itself.

For Docusaurus projects, the module paths are already configured. If you're using TypeScript in a custom plugin or theme, ensure your`tsconfig.json` includes`@docusaurus/*` in its module resolution so TypeScript can find the type definitions.

## Limitations

- **Generated modules are build-time only**, The`@generated/*` modules are created during the build process. They aren't available for inspection until after you run`docusaurus build` or`docusaurus start`.

- **Type definitions follow the Docusaurus version**, The types are tied to your installed Docusaurus version. If you upgrade Docusaurus, some types may change.

- **Plugin types depend on plugin availability**, If you reference a plugin's global data through`useGlobalData()`, that data only exists if the plugin is installed and configured.

- **Runtime is still required**, Type definitions don't eliminate the need for proper error handling. Typed code can still reference missing data at runtime if you're not careful about checking for undefined values.

---

**Related topics:**
- Site Scaffolding, Create a new Docusaurus project with TypeScript support
- MDX & Markdown Authoring, Write content with TypeScript-aware components
- Logging & Diagnostics, Debug TypeScript errors and type issues
