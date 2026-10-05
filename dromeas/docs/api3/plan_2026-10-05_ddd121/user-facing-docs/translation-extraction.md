# Translation extraction

Translation Extraction finds text in your project that is marked for translation and collects it into a single, organized list. This list is the foundation for internationalization and lets you deliver your site in multiple languages.

**Who it is for**

- Developers who write code that displays text to users.
- Translators and localization managers who need a complete, reliable list of every message a project can show.

**What it does**

When you prepare your site for translation, the tool scans your source files and looks for text wrapped in two supported patterns:

- A`<Translate>` component used in markup.
- A`translate()` function used in JavaScript or TypeScript code.

Each marked piece of text is pulled out and recorded with:

- An identifier, either provided by you or generated from the message text.
- The message itself.
- An optional description that gives translators context.

**Ways to mark text for extraction**

**Use the`<Translate>` component**

You can add a`<Translate>` component around text in your markup. It supports:

- An`id` attribute for a stable identifier.
- A`description` attribute to explain the text.
- The text itself, placed as the component's content.

If you don't supply an`id`, the message text is used as the identifier. A`<Translate>` component with no child content must include an`id`.

Example:
```
<Translate id="greeting" description="A friendly greeting">
  Welcome to our site
</Translate>
```

**Use the`translate()` function**

You can call a`translate()` function in your code and pass a single object with these properties:

-`message`: the text to be translated.
-`id`: a stable identifier (optional).
-`description`: translator context (optional).

The function accepts exactly one or two arguments. The first argument must be an object with the properties listed above.

Example:
```
translate({
  message: "Welcome to our site",
  id: "greeting",
  description: "A friendly greeting"
})
```

**What can be extracted**

- Text inside`<Translate>` that is a static string or an expression that can be fully evaluated before your site is built.
- Static`id` and`description` values.
- Text from`translate()` calls that use a fixed object with static values.
- Multiple`<Translate>` components and`translate()` calls across all your source files are collected into one translation inventory.

**What cannot be extracted**

- Text built dynamically at runtime, such as variable values, function results, or string concatenation that cannot be resolved at build time.
- A`<Translate>` component whose content mixes static and dynamic parts, or contains nested components or complex expressions.
- A`translate()` call with more than two arguments or where the first argument is not a statically evaluable object.
- Properties like`id` or`description` that are set dynamically or cannot be determined before the build completes.

When the tool finds content it cannot safely extract, it issues a warning. The warning shows the source file name, the line number where the problem occurs, and the problematic code snippet. It also provides examples of the correct pattern so you can fix the issue.

**Benefits**

- Captures every marked string across your project in one place, so no text is missed.
- Produces a translation inventory that plugs directly into localization workflows.
- Gives early feedback on unsupported dynamic values, allowing you to fix them before translation begins.
- Keeps source text close to the code where it's used while the inventory is built automatically.
- Supports both JSX component syntax and function-call syntax, letting you choose what fits your codebase style.

**Limits**

- Only text explicitly marked with`<Translate>` or`translate()` is extracted. Unmarked text in your source files is ignored completely.
- All values—message,`id`, and`description`—must be static and known at build time. Dynamic values are reported as warnings and excluded from the translation inventory.
- A`<Translate>` component with no child text requires an`id` attribute; otherwise the tool issues a warning and skips that entry.
- Empty text nodes and whitespace-only content in`<Translate>` components are filtered out to avoid translation of formatting artifacts.
- The extractor requires that you import from`@docusaurus/Translate` in order to recognize`<Translate>` or`translate` in your file.

**Mixing both patterns**

You can use the`<Translate>` component and`translate()` function together in the same codebase. Both produce the same output format and are processed by the same extraction system. Each entry in the translation inventory can carry an optional`description` to provide translators with context about where or how the text is used.

The extraction process handles JSX files, TypeScript files, and JavaScript files, automatically recognizing the correct syntax for each file type based on its extension.
