# How to embed interactive react components in MDX

Embed React components directly in your MDX files to create interactive, dynamic documentation. MDX combines Markdown's simplicity with React's power, letting you add interactive elements alongside your written content.

## Goal

You want to add interactive React components (buttons, forms, charts, toggles, or custom widgets) to your documentation pages written in MDX format.

## Prerequisites

- You have a Docusaurus site set up with MDX support enabled
- You're working with`.mdx` files in your documentation
- You have React components ready to embed or you're comfortable creating simple components
- You understand basic Markdown and React syntax

## Steps

### 1. Create or prepare your react component

Before embedding, ensure you have a React component file. Save it in a location accessible to your documentation, such as`src/components/` or`docs/components/`.

Create a simple example component:

```jsx
// src/components/InteractiveButton.jsx
import React, { useState } from 'react';

export default function InteractiveButton() {
  const [count, setCount] = useState(0);

  return (
    <div style={{ padding: '16px', border: '1px solid #ddd', borderRadius: '8px' }}>
      <p>Button clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

### 2. Import the component at the top of your MDX file

Add an import statement at the beginning of your`.mdx` file:

```mdx
import InteractiveButton from '@site/src/components/InteractiveButton';
```

The`@site/` prefix refers to your site root directory. Adjust the path to match your component's location.

### 3. Use the component in your MDX content

Once imported, use the component like you would in any React file by writing it as a JSX element:

```mdx
# My Documentation Page

Here's some regular Markdown content explaining a feature.

<InteractiveButton />

And here's more Markdown after the interactive component.
```

### 4. Pass props to customize behavior

You can pass props to your components to customize them for different contexts:

```mdx
import Counter from '@site/src/components/Counter';

<Counter initialValue={5} step={2} />

<Counter initialValue={10} step={1} />
```

Update your component to accept and use these props:

```jsx
export default function Counter({ initialValue = 0, step = 1 }) {
  const [count, setCount] = useState(initialValue);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + step)}>
        Increment by {step}
      </button>
    </div>
  );
}
```

### 5. Access document metadata in your component (optional)

Use the`useDocusaurusContext` hook to access site configuration and metadata:

```jsx
import { useDocusaurusContext } from '@docusaurus/Docusaurus';

export default function SiteAwareComponent() {
  const { siteConfig } = useDocusaurusContext();

  return <div>Welcome to {siteConfig.title}</div>;
}
```

### 6. Style your component

Use inline styles, CSS modules, or Docusaurus's CSS variable system:

```jsx
import styles from './MyComponent.module.css';

export default function MyComponent() {
  return (
    <div className={styles.container}>
      Content with custom styles
    </div>
  );
}
```

Or use Docusaurus theme variables:

```jsx
export default function ThemedComponent() {
  return (
    <div
      style={{
        color: 'var(--ifm-color-primary)',
        padding: 'var(--ifm-spacing-horizontal)',
      }}
    >
      This respects your site's theme colors
    </div>
  );
}
```

## Expected result

Your documentation page renders with:
- Regular Markdown text displayed normally
- React components rendered as interactive elements
- Components receiving and responding to user interactions
- Props correctly passed and used by components
- Proper styling applied from your component files

When you run`docusaurus build` or`docusaurus start`, the MDX loader processes your`.mdx` files, converts them to valid JavaScript that includes your imported React components, and renders everything in the browser.

## Common issues

**"Component is not defined" error**

Verify your import path is correct. Use`@site/` for imports relative to your site root. Check that the file exists and the export name matches.

**Component doesn't render or appears blank**

Ensure the component exports a valid React element as the default export. Check browser console for error messages. Verify that any state hooks or effects in your component don't have errors.

**Props aren't being passed correctly**

Make sure prop names match exactly between your MDX usage and component definition. For boolean props, use`<Component active={true} />` rather than`<Component active />`. Check the component's props destructuring syntax.

**Styling doesn't apply**

Confirm CSS files are imported correctly in your component. If using CSS modules, verify the module file exists in the same directory. For CSS variables, ensure they're defined in your Docusaurus theme configuration.

**Import path issues in nested MDX files**

For MDX files in subdirectories like`docs/guides/setup.mdx`, the`@site/` prefix still points to your site root, not relative to the MDX file. Use`@site/src/components/MyComponent` consistently.

## Related

- MDX & Markdown Authoring
- How to write Markdown content with front matter
- How to render Mermaid diagrams
