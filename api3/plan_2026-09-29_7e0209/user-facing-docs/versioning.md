# Versioning

Docusaurus versioning lets you keep documentation for every release of your project available in one place. Your readers can switch between versions without leaving the documentation, and you can update the in-progress version without affecting published documentation.

## What versioning does

When you publish a new release, you can freeze the current state of your documentation into a versioned snapshot. That snapshot stays exactly as readers saw it at release time. Meanwhile, the live documentation continues to change as you write updates for the next release.

Versioning is useful for any project that needs to support older releases. It lets users of an older product version find instructions that match the version they have installed.

## Create a versioned release

You create a version by running a single command, such as:

```bash
npm run docusaurus docs:version 1.0
```

This command performs two actions:

- It copies the current documentation into a frozen snapshot labeled **1.0**.
- It keeps the original documentation available as the **Next** version, where you can continue writing updates.

After you create a version, the new version appears in the version dropdown and becomes the default documentation that readers see.

## Maintain multiple versions

Each versioned snapshot contains the full text of your documentation from the moment that version was created. Readers can select any available version from the version dropdown to read the documentation that applied to that release.

You can create additional versions at any time with the same command and a new version label, such as:

```bash
npm run docusaurus docs:version 2.0
```

Every version remains accessible unless you remove it. The most recently created version becomes the new default for readers, while **Next** continues to contain unreleased documentation.

## Understand **current** and **Next** versions

Docusaurus uses two concepts to distinguish released and unreleased documentation:

- **Current** is the most recently frozen release snapshot. For example, after running the command with `2.0`, version **2.0** becomes the current version. This is the default version readers see when they open the documentation.
- **Next** is the live, editable documentation. It is displayed with a **Next** label in the version dropdown and typically appears first in the list so that contributors can preview upcoming content.

Readers can switch between **Next** and any numbered release at any time.

## Add version labels

Version labels are the names shown in the version dropdown. By default, each snapshot uses the version number you provide when you create it, such as `1.0` or `2.0`. The **Next** version has a special label that marks it as unreleased.

You can choose the labels that appear in the dropdown. For example, you might label a version `1.0.0` to match a specific release tag, or use a friendly name that your users recognize.

## Keep separate sidebars for each version

Each version can have its own documentation sidebar. When you create a version, the current sidebar structure is frozen along with the docs. This means older documentation can keep an older navigation layout while the **Next** version uses a newer sidebar.

You can also choose to reuse the same sidebar across versions. Using versioned sidebars is helpful when a release adds new pages, removes old ones, or reorganizes documentation into new categories.

## What versioning applies to

Docusaurus versioning applies only to documentation. It does not version blog posts, standalone pages, or other site content.

If your site contains release announcements or changelogs, versioning does not automatically create snapshots for those areas. You can still maintain version-specific documentation links and guides manually, but the built-in versioning workflow is limited to the docs section.