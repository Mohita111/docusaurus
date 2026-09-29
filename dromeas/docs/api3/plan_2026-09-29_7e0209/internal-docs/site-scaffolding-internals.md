# Site Scaffolding Internals

This document describes the internal architecture, module boundaries, data flow, and extension points of the Docusaurus site scaffolding system implemented in `packages/create-docusaurus`.

## Overview

The scaffolding system generates a new Docusaurus site from a template, a Git repository, or a local directory. It is implemented as a lightweight Node.js CLI with minimal dependencies to keep installation fast.

**Core source files:**

| File | Responsibility |
|------|----------------|
| `packages/create-docusaurus/bin/index.js` | CLI entry point and argument parsing |
| `packages/create-docusaurus/src/index.ts` | Orchestration logic (`init`) and source resolution |
| `packages/create-docusaurus/src/commands.ts` | Subprocess execution and package manager detection |
| `packages/create-docusaurus/src/constants.ts` | Shared types and constant values |
| `packages/create-docusaurus/src/prompts.ts` | Interactive prompt helpers |
| `packages/create-docusaurus/src/utils.ts` | Filesystem and package.json utilities |
| `packages/create-docusaurus/templates/` | Built-in site templates |

## Module breakdown

### CLI entry point: `bin/index.js`

The executable at `packages/create-docusaurus/bin/index.js` is the process entry point. It performs three tasks before delegating to the library:

1. **Node version check** using `semver.satisfies()` against `packageJson.engines.node`.
2. **Argument parsing** with `commander`.
3. **Dynamic import** of the compiled `lib/index.js` to keep startup time low.

Command signature:

```text
create-docusaurus [siteName] [template] [rootDir]
```

Options:

| Option | Short | Description |
|--------|-------|-------------|
| `--package-manager <manager>` | `-p` | One of `yarn`, `npm`, `pnpm`, `bun` |
| `--skip-install` | `-s` | Skip dependency installation after scaffolding |
| `--typescript` | `-t` | Use the TypeScript template variant |
| `--javascript` | `-j` | Use the JavaScript template variant |
| `--git-strategy <strategy>` | `-g` | One of `deep`, `shallow`, `copy`, `custom` |

The action handler calls `init()` asynchronously:

```js
.action((siteName, template, rootDir, options) =>
  import('../lib/index.js').then(({default: init}) =>
    init(path.resolve(rootDir ?? '.'), siteName, template, options),
  ),
);
```

Unhandled rejections are logged with `util.inspect()` and cause a non-zero exit.

### Constants: `src/constants.ts`

All shared types and runtime constants are defined here.

**Package managers.** `PackageManager` is derived from the keys of `LockfileNames`, which also enforces an intentional ordering used for lock file detection:

```ts
export const LockfileNames = {
  npm: 'package-lock.json',
  yarn: 'yarn.lock',
  pnpm: 'pnpm-lock.yaml',
  bun: 'bun.lockb',
};

export type PackageManager = keyof typeof LockfileNames;

export const PackageManagers = Object.keys(LockfileNames) as PackageManager[];
```

**Git clone strategies.** The `GitCloneStrategies` tuple is the source of truth for both validation and the interactive prompt choices:

```ts
export const GitCloneStrategies = [
  'deep',
  'shallow',
  'copy',
  'custom',
] as const;

export type GitCloneStrategy = (typeof GitCloneStrategies)[number];
```

**Template descriptor.** Each built-in template is described by a name, its filesystem path, and an optional TypeScript variant path:

```ts
export type Template = {
  name: string;
  path: string;
  tsVariantPath: string | undefined;
};
```

**Source union type.** `Source` is the discriminated union that all source-resolution functions produce. It is the central data structure passed to the copy/clone phase:

```ts
export type Source =
  | {
      type: 'template';
      template: Template;
      language: 'javascript' | 'typescript';
    }
  | {
      type: 'git';
      url: string;
      strategy: GitCloneStrategy;
    }
  | {
      type: 'local';
      path: string;
    };
```

`DefaultPackageManager` is set to `npm` and serves as the fallback when global installation and `--skip-install` prevent reliable detection.

### Command execution: `src/commands.ts`

This module wraps `cross-spawn` to handle Windows command resolution (`yarn.cmd` vs `yarn`) and for color-flag handling.

**`runCommand(command, args, options)`** splits a command string into a real command plus base args, then merges them with the supplied args. It resolves with the child process exit code or rejects on spawn failure. Standard I/O is ignored by default (`{stdio: 'ignore'}`).

**Package manager detection** is performed by checking whether each known package manager responds to `--version`:

```ts
export async function getAvailablePackageManagers(): Promise<PackageManager[]> {
  const list = await Promise.all(
    PackageManagers.map(async (name) => {
      return (await hasPackageManager(name)) ? name : null;
    }),
  );
  return list.filter((item) => item !== null);
}
```

**Installation command** varies by package manager. Bun and Yarn use a single bare command, while npm and pnpm receive an install subcommand and a forced color flag:

```ts
const installCommand =
  pkgManager === 'yarn'
    ? 'yarn'
    : pkgManager === 'bun'
      ? 'bun install'
      : `${pkgManager} install --color always`;
```

Color output is forced by adding `FORCE_COLOR: '1'` to the spawn environment when `supportsColor.stdout` is truthy.

**Git clone** maps strategy values to a concrete command. `shallow` and `copy` both clone with `--recursive --depth 1`; `custom` prompts for an arbitrary command; `deep` and the default invoke plain `git clone`:

```ts
async function getGitCloneCommand(
  gitStrategy: GitCloneStrategy,
): Promise<string> {
  switch (gitStrategy) {
    case 'shallow':
    case 'copy':
      return 'git clone --recursive --depth 1';
    case 'custom': {
      return askForCustomGitCloneCommand();
    }
    case 'deep':
    default:
      return 'git clone';
  }
}
```

The exported `runGitCloneCommand()` invokes `getGitCloneCommand()` and appends `[source.url, dest]` as positional arguments.

### Main orchestration: `src/index.ts`

The default export `init(rootDir, reqName, reqTemplate, cliOptions)` orchestrates the full scaffolding pipeline.

#### Signature and options

```ts
export default async function init(
  rootDir: string,
  reqName?: string,
  reqTemplate?: string,
  cliOptions: CLIOptions = {},
): Promise<void>
```

`CLIOptions` combines language flags, package manager selection, `skipInstall`, and `gitStrategy`:

```ts
type CLIOptions = LanguagesOptions & {
  packageManager?: PackageManager;
  skipInstall?: boolean;
  gitStrategy?: GitCloneStrategy;
};
```

#### Scaffolding pipeline

The `init` function follows a strictly ordered pipeline:

1. **Read built-in templates** and **resolve site name** in parallel. `readTemplates()` scans `../templates` relative to `import.meta.url`, excluding hidden files, `README*`, TypeScript variants, and the `shared` directory. Templates are sorted so `classic` appears first.

2. **Resolve the source.** `getSource()` delegates to three paths:
   - `getUserProvidedSource()` when a CLI template argument exists. This checks for Git URLs (`https://` or `git@` prefix), existing local paths, or falls back to built-in template lookup.
   - `askTemplateChoice()` shows an interactive prompt for built-in templates plus `Git repository` and `Local template`.
   - `askGitRepositorySource()` and `askLocalSource()` collect the relevant details interactively.

3. **Copy or clone.** For `type === 'git'`, `runGitCloneCommand()` is called and, for `copy` strategy, the `.git` directory is removed. For `type === 'template'`, `copyTemplate()` first copies the `shared` directory and then overlays the chosen template variant. For `type === 'local'`, `fs.cp()` copies the local directory recursively.

4. **Update package.json** through `updatePkg()` with `name`, `version: '0.0.0'`, and `private: true`. The `name` field is normalized with `siteNameToPackageName()`.

5. **Normalize `.gitignore`.** If a `gitignore` file exists but `.gitignore` does not, it is renamed. Any remaining `gitignore` file is removed.

6. **Resolve package manager.** `getPackageManager()` checks in order: an existing lock file in the destination, the explicit CLI option, a lock file in the current working directory, the `npm_config_user_agent` environment variable, and finally an interactive prompt or the default.

7. **Install dependencies unless `--skip-install`** is set. The working directory is changed to `dest` with `process.chdir()`, then `runPackageManagerInstallCommand()` executes the appropriate install command. If installation fails, the site directory is preserved and the user receives retry instructions.

8. **Print help.** `printPackageManagerHelp()` displays commands for starting, building, serving, and deploying the site.

#### Key internal functions

**`readTemplates()`** discovers templates from the filesystem. TypeScript variants are detected by a `-typescript` suffix:

```ts
const tsVariantPath = path.join(
  templatesDir,
  `${name}${typeScriptTemplateSuffix}`,
);
return {
  name,
  path: path.join(templatesDir, name),
  tsVariantPath: (await pathExists(tsVariantPath))
    ? tsVariantPath
    : undefined,
};
```

**`copyTemplate()`** layers the `shared` directory and the variant-specific directory:

```ts
await fs.cp(path.join(templatesDir, 'shared'), dest, {
  recursive: true,
});

const sourcePath =
  language === 'typescript' ? template.tsVariantPath! : template.path;

await fs.cp(sourcePath, dest, {
  recursive: true,
  filter: async (filePath) => !(await fs.lstat(filePath)).isSymbolicLink(),
});
```

Symlinks are filtered because published npm packages no longer contain them; the filter exists only to prevent errors during local testing.

**`getUserProvidedSource()`** returns one of three `Source` variants. Git URL detection is based only on prefix matching:

```ts
function isValidGitRepoUrl(gitRepoUrl: string): boolean {
  return ['https://', 'git@'].some((item) => gitRepoUrl.startsWith(item));
}
```

If the provided template string is not a URL and not an existing local path, it is treated as a built-in template name via `getTemplateSource()`.

**`getSiteName()`** validates the requested name. For `siteName === '.'`, it checks that the target directory is empty; for any other name, it checks that the destination path does not already exist. When invoked interactively, it uses `prompts` with `initial: 'website'`.

### Prompt helpers: `src/prompts.ts`

Two prompt helpers are defined here to break a potential circular dependency with `index.ts`.

**`askPreferredLanguage()`** presents JavaScript and TypeScript choices, both styled with `logger.bold()`. If the selection is canceled, `process.exit(0)` terminates the process.

**`askForCustomGitCloneCommand()`** prompts for a custom Git clone command. On cancel, it falls back to `git clone` rather than terminating:

```ts
return command ?? 'git clone';
```

The message explains that the repository URL and destination directory will be automatically appended.

### Utility functions: `src/utils.ts`

**`siteNameToPackageName(siteName)`** converts a display site name into a valid npm package name using a regex-based kebab-case conversion. It intentionally avoids external dependencies like `lodash`. The regex handles acronyms, camelCase boundaries, and digits:

```ts
const match = siteName.match(
  /[A-Z]{2,}(?=[A-Z][a-z]+\d*|\b|_)|[A-Z]?[a-z]+\d*|[A-Z]|\d+/g,
);
if (match) {
  return match.map((x) => x.toLowerCase()).join('-');
}
return siteName;
```

**`updatePkg(pkgPath, obj)`** reads the destination `package.json`, shallow-merges the supplied properties, and writes back with two-space indentation and a trailing newline. A common failure point; the caller wraps it in a `try/catch` with a descriptive error message.

**`pathExists(filePath)`** wraps `fs.access()` in a boolean promise without requiring `fs-extra` as a dependency:

```ts
export async function pathExists(filePath: string): Promise<boolean> {
  return fs
    .access(filePath, fs.constants.F_OK)
    .then(() => true)
    .catch(() => false);
}
```

**`printPackageManagerHelp()`** formats the post-installation success message. It adapts the command syntax based on the package manager. npm and Bun omit the `run` subcommand for lifecycle scripts, while Yarn and pnpm include it:

```ts
const useNpm = pkgManager === 'npm';
const useBun = pkgManager === 'bun';
const run = useNpm || useBun ? 'run ' : '';
```

## Data flow

```
bin/index.js
    │  commander parses CLI options
    ▼
src/index.ts init(rootDir, siteName, template, cliOptions)
    │
    ├── readTemplates() ──► Template[] from templates/ directory
    ├── getSiteName() ──► validated site name (interactive if required)
    │
    ├── getSource(reqTemplate, templates, cliOptions)
    │       ├── getUserProvidedSource() ──► CLI template, git URL, local path, or built-in
    │       ├── askTemplateChoice() ──► interactive selection
    │       ├── askGitRepositorySource() ──► repository URL + clone strategy
    │       └── askLocalSource() ──► local directory path
    │       Returns: Source (discriminated union)
    │
    ├── copy/clone phase based on Source
    │       ├── type 'git'   ──► runGitCloneCommand()
    │       ├── type 'template' ──► copyTemplate() (shared + variant)
    │       └── type 'local'  ──► fs.cp()
    │
    ├── updatePkg(dest/package.json, {name, version, private})
    ├── normalize .gitignore
    │
    ├── getPackageManager(dest, cliOptions)
    │       ├── findPackageManagerFromLockFile(dest)
    │       ├── explicit cliOptions.packageManager
    │       ├── findPackageManagerFromLockFile('.')
    │       ├── findPackageManagerFromUserAgent()
    │       └── askForPackageManagerChoice() / default
    │
    ├── runPackageManagerInstallCommand() unless --skip-install
    │
    └── printPackageManagerHelp()
```

## Extension points

### Custom built-in templates

New templates are auto-discovered by `readTemplates()` from the `templates/` directory. To add a template:

1. Create a directory under `packages/create-docusaurus/templates/` with the template name (for example, `my-template/`).
2. Optionally create a TypeScript variant at `my-template-typescript/`. The detection logic checks for this suffix:

```ts
const tsVariantPath = path.join(
  templatesDir,
  `${name}${typeScriptTemplateSuffix}`,
);
```

3. Templates are sorted with `classic` first (using `recommendedTemplate = 'classic'`). To control ordering, modify the `sort()` comparator in `readTemplates()`.

The `shared/` directory is copied into every new site before the template-specific overlay, making it the canonical place for files common to all templates. The `shared/` and `README*` entries are explicitly excluded from discovery.

### Custom Git clone strategies

The supported clone strategies are defined in `src/constants.ts` as `GitCloneStrategies`. To add a new strategy:

1. Add the value to the `GitCloneStrategies` tuple.
2. Add a mapping case in `getGitCloneCommand()` in `src/commands.ts`.
3. Optionally add an interactive prompt choice in `askGitRepositorySource()` in `src/index.ts`.

The `custom` strategy is an existing example: it delegates to `askForCustomGitCloneCommand()` in `src/prompts.ts` and defaults to `git clone` on cancellation.

### Local templates

Local templates are directories referenced through the CLI third positional argument (`[template]`) or the interactive `Local template` option. `getUserProvidedSource()` converts an existing local path into `{type: 'local', path: path.resolve(reqTemplate)}`. The `.gitignore` normalization behavior is applied to local templates as well, enabling Git-hosted template directories to ship a `gitignore` file that is renamed to `.gitignore`.

### Template structure: classic

The `classic` template contains a `docusaurus.config.js` and `sidebars.js` that define defaults for every generated site.

**`docusaurus.config.js`** declares the presets, theme configuration, and future flag:

```js
const config = {
  title: 'My Site',
  tagline: 'Dinosaurs are cool',
  future: { v4: true },
  url: 'https://your-docusaurus-site.example.com',
  baseUrl: '/',
  // ...
  presets: [['classic', { docs: { sidebarPath: './sidebars.js' }, /* ... */ }]],
  themeConfig: { /* navbar, footer, prism */ },
};
```

**`sidebars.js`** defines an autogenerated sidebar from the `docs` directory:

```js
const sidebars = {
  tutorialSidebar: [{type: 'autogenerated', dirName: '.'}],
};
```

These are copied into new sites and customized through the `updatePkg()` phase, though their contents are not modified programmatically.

## Error handling strategy

The pipeline uses three distinct error-handling mechanisms:

1. **Validation failures before copying** throw plain `Error` objects with interpolated `logger` messages. For example, an invalid package manager choice:

```ts
throw new Error(
  `Invalid package manager choice ${packageManager}. Must be one of ${PackageManagers.join(
    ', ',
  )}`,
);
```

2. **Copy failures during the scaffolding phase** throw errors with a `cause` property connecting the underlying filesystem error:

```ts
throw new Error(
  logger.interpolate`Copying Docusaurus template name=${source.template.name} failed!`,
  {cause: err},
);
```

3. **Dependency installation failures** log an error and exit with code 0 (not a failure code) because the site directory has already been created successfully. The user is shown commands to retry installation manually.

The CLI entry point also registers a global `unhandledRejection` handler that logs the full error via `util.inspect()` and exits with code 1.

## Design constraints

The `src/index.ts` module carries an explicit dependency-weight note:

```ts
// KEEP DEPENDENCY SMALL HERE!
// create-docusaurus CLI should be as lightweight as possible
```

This drives several implementation choices:

- `cross-spawn` is used instead of `execa` to avoid a heavier dependency.
- `siteNameToPackageName()` implements kebab-case conversion manually to avoid `lodash`.
- `pathExists()` uses `fs.access()` instead of `fs-extra`.
- The CLI uses `commander` for parsing and a dynamic `import('../lib/index.js')` to defer loading the scaffolding logic until after argument parsing.
- The only direct third-party runtime dependencies are `commander`, `semver`, `prompts`, `cross-spawn`, and `supports-color`, plus the internal `@docusaurus/logger`.