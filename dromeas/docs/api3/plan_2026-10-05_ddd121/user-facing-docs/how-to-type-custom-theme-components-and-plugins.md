# How to type custom theme components and plugins

You can use TypeScript to add type safety to custom theme components and plugins in Docusaurus. This guide walks you through importing and using the pre-defined module type aliases that give you full type checking for your theme and plugin code.

## Prerequisites

- Your project uses TypeScript (a`.ts` or`.tsx` file in your theme or plugin)
- You have access to the Docusaurus module type aliases package (`@docusaurus/module-type-aliases`)
- You understand basic TypeScript syntax and how to import modules

## How module type aliases work

When you build your Docusaurus site, special generated modules become available that provide type information for your theme and plugins. These modules include:

-`@generated/docusaurus.config`, your site configuration
-`@generated/globalData`, global data from all plugins
-`@generated/routes`, all site routes
-`@theme/Layout`, the main layout component
-`@docusaurus/useDocusaurusContext`, hook to access site context
- And many others for common theme and plugin patterns

TypeScript needs declarations for these modules so your editor can show you available types and catch errors before you build.

## Typed theme component example

Here's how to create a typed custom theme component that uses Docusaurus types:

1. **Create your theme component file** in your project's theme folder, for example at`src/theme/CustomComponent.tsx`.

2. **Import the types you need** from the Docusaurus module aliases:

```typescript
   import type {ReactNode} from 'react';
   
   interface Props {
     readonly children?: ReactNode;
   }
   
   export default function CustomComponent(props: Props): ReactNode {
     return <div>{props.children}</div>;
   }
   ```

3. **Use Docusaurus hooks** with full type safety:

```typescript
   import {useDocusaurusContext} from '@docusaurus/useDocusaurusContext';
   import type {ReactNode} from 'react';
   
   export default function CustomComponent(): ReactNode {
     const {siteConfig} = useDocusaurusContext();
     return <h1>{siteConfig.title}</h1>;
   }
   ```

   The`useDocusaurusContext()` hook returns a`DocusaurusContext` type that includes`siteConfig`,`i18n`, and other properties, all typed correctly.

## Typed plugin example

When building a plugin with TypeScript, you can import generated types for your site's configuration and data:

1. **Import the configuration module** in your plugin code:

```typescript
   import config from '@generated/docusaurus.config';
   import type {DocusaurusConfig} from '@docusaurus/types';
   
   export function myPlugin(): void {
     const siteTitle: string = config.title;
     // TypeScript knows config has a 'title' property of type string
   }
   ```

2. **Access global plugin data** with types:

```typescript
   import globalData from '@generated/globalData';
   import type {GlobalData} from '@docusaurus/types';
   
   const data: GlobalData = globalData;
   // TypeScript ensures globalData matches the GlobalData shape
   ```

## Available typed modules

The following modules come with pre-defined types:

| Module | Type | Use case |
|--------|------|----------|
|`@generated/docusaurus.config` |`DocusaurusConfig` | Access your site configuration |
|`@generated/globalData` |`GlobalData` | Get data from all plugins |
|`@generated/site-metadata` |`SiteMetadata` | Access site metadata |
|`@generated/routes` |`RouteConfig[]` | Work with site routes |
|`@theme/Layout` |`Props {children?: ReactNode}` | Extend or wrap the layout |
|`@theme/Root` |`Props {children: ReactNode}` | Add a root provider |
|`@docusaurus/useDocusaurusContext` |`DocusaurusContext` | Access site context in components |
|`@docusaurus/useRouteContext` |`PluginRouteContext` | Get the current route context |
|`@docusaurus/Head` |`Props` (extends`HelmetProps`) | Manage document head tags |
|`@docusaurus/Link` |`Props` | Create typed internal links |

## Check types in your editor

Once you've set up TypeScript and imported the types, your editor shows:

- **IntelliSense suggestions** when you type properties like`config.` or`useDocusaurusContext().`
- **Red squiggles** if you try to access a property that doesn't exist
- **Hover tooltips** showing the expected type of each property
- **Build-time errors** caught by the TypeScript compiler before you deploy

## Common issues and solutions

**Issue: "Cannot find module '@generated/docusaurus.config'"**

This means your project hasn't been built yet, or the TypeScript compiler can't see the generated types. Run`npm run build` to generate the modules, then rebuild your TypeScript.

**Issue: "Property does not exist on type DocusaurusContext"**

Check the module type aliases source file to see the exact properties available on each type. The property name or spelling may differ from what you expected.

**Issue: Type errors after updating Docusaurus**

The module type aliases are updated with each Docusaurus release. Update your`@docusaurus/module-type-aliases` package to match your Docusaurus version:`npm install @docusaurus/module-type-aliases@latest`.

## Result

After following these steps, you have:

- Full TypeScript support for custom theme components
- Type-safe access to Docusaurus hooks, configuration, and global data
- Editor autocomplete for all Docusaurus APIs
- Compile-time error checking before you deploy

---

**Related pages:**

- TypeScript Module Aliases, Overview of the module aliases feature
- System Requirements and Prerequisites, Required Node.js and TypeScript versions
