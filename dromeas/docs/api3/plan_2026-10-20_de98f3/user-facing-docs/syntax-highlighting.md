# Syntax highlighting

Syntax highlighting is the color-coded display of code in the code editor. Keywords, strings, punctuation, and other parts of your code appear in distinct colors so you can tell them apart at a glance. It is built for developers who read, write, and review code in more than one language.

## What it does

- Colors your code automatically as you type or paste it into the editor
- Supports **SQL** and **Markdown** content directly in the editor
- Keeps snippets readable without any extra tools or setup

You do not need to install or configure anything to use syntax highlighting. It is already part of the snippet editor.

## Who it helps

Syntax highlighting helps anyone working with code snippets, including:

- Developers who switch between SQL queries and documentation-style Markdown
- People reviewing saved code and looking for a specific line or structure
- Anyone who wants to reduce eye strain while reading dense code

## Benefits

With color-coded display, you can:

- Recognize code structure instantly through distinct colors
- Scan long snippets more quickly
- Reduce eye strain when reading or editing for extended periods
- Spot keywords, values, and punctuation without reading every character

Syntax highlighting turns a wall of text into a readable, scannable view of your code.

Figure: How syntax highlighting appears in the editor

```mermaid
graph LR
    A[Open the code editor] --> B[Write or paste code]
    B --> C[Editor identifies language details]
    C --> D[Code displays with color coding]
    D --> E[Read and edit with less effort]
```

## Supported languages

The built-in highlighter includes dedicated support for:

- **SQL**
- **Markdown**

These languages are covered by a lightweight, built-in highlighter that keeps the experience simple and fast. Other languages may not receive the same depth of color coding.

## Automatic language detection

When the application can recognize the language, highlighting applies automatically as you type. You do not need to select a language or change any settings to see the color-coded display.

Automatic recognition is most applicable to the supported languages, SQL and Markdown. If the editor cannot clearly identify the language, color coding may be limited.

## Built in and lightweight

Syntax highlighting is built into the application, not added as an external library. This keeps the editor lightweight and responsive, so color coding appears quickly without slowing down your work.

Because the highlighter is intentionally simple, it focuses on reliable SQL and Markdown support rather than trying to cover every possible language.

## What to keep in mind

- Highlighting is most complete for **SQL** and **Markdown**
- The color palette and depth of highlighting are intentionally simple
- Other languages may appear with less color detail or no automatic recognition
- You do not need to configure or enable anything to get started

These boundaries keep the editor fast and avoid the overhead of a larger external highlighting tool.
