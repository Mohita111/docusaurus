# Site Scaffolding Internals

This document describes the internal architecture of `@docusaurus/create-docusaurus`, the Docusaurus site scaffolding CLI. It covers entry points, source resolution, template copying, package manager detection, command execution, and extension points.

## Architecture overview

The scaffolding flow is:

1. The CLI entry point parses arguments and options.
2. `init` resolves the site name and available templates.
3. `getSource` converts the requested template into a `Source` object.
4. The selected source is materialized into the destination directory.
5. `package.json` is updated with the generated package name.
6. The package manager is selected.
7. Dependencies are installed unless `--skip-install` is used.
8. Help text is printed.

Core modules:

| Module | Path | Responsibility |
| --- | --- | --- |
| CLI entry point | `packages/create-docusaurus/bin/index.js` | Arg parsing, Node.js version check |
| Scaffolding orchestrator | `packages/create-docusaurus/src/index.ts` | `init`, source resolution, copying, install flow |
| Command execution | `packages/create-docusaurus/src/commands.ts` | Cross-platform command spawning, package manager commands |
| Shared constants | `packages/create-docusaurus/src/constants.ts` | Package managers, lockfiles, `Source`, `Template` types |
| User prompts | `packages/create-docusaurus/src/prompts.ts` | Language and custom clone prompt |
| Utilities | `packages/create-docusaurus/src/utils.ts` | Package name conversion, package.json update, help output |
| Built-in templates | `packages/create-docusaurus/templates/` | Classic template, shared assets, TypeScript variants |

---

## CLI entry point

File: `packages/create-docusaurus/bin/index.js`

The entry point is an ESM Node.js script. It imports its own `package.json` through `createRequire`:

```ts
const packageJson = createRequire(import.meta.url)('../package.json');
const requiredVersion = packageJson.engines.node;
```

It rejects unsupported Node.js versions:

```ts
if (!semver.satisfies(process.version, requiredVersion)) {
  logger.error('Minimum Node.js version not met :(');
  // ...
  process.exit(1);
}
```

The CLI is defined with Commander:

```ts
program
  .arguments('[siteName] [template] [rootDir]')
  .option('-p, --package-manager <manager>', 'The package manager used to install dependencies. One of yarn, npm, pnpm, and bun.')
  .option('-s, --skip-install', 'Do not run package manager immediately after scaffolding')
  .option('-t, --typescript', 'Use the TypeScript template variant')
  .option('-j, --javascript', 'Use the JavaScript template variant')
  .option('-g, --git-strategy <strategy>', '...')
  .description('Initialize website.')
  .action((siteName, template, rootDir, options) =>
    import('../lib/index.js').then(({default: init}) =>
      init(path.resolve(rootDir ?? '.'), siteName, template, options),
    ),
  );
```

The action lazy-loads the compiled scaffolding module from `../lib/index.js`. The default export of that module is the `init` function.

Unhandled promise rejections are caught globally and printed with `util.inspect`:

```ts
process.on('unhandledRejection', (error) => {
  logger.error(inspect(error));
  process.exit(1);
});
```

---

## Scaffolding entry function

File: `packages/create-docusaurus/src/index.ts`

The main function is:

```ts
export default async function init(
  rootDir: string,
  reqName?: string,
  reqTemplate?: string,
  cliOptions: CLIOptions = {},
): Promise<void>
```

`CLIOptions` is defined as:

```ts
type CLIOptions = LanguagesOptions & {
  packageManager?: PackageManager;
  skipInstall?: boolean;
  gitStrategy?: GitCloneStrategy;
};

type LanguagesOptions = {
  javascript?: boolean;
  typescript?: boolean;
};
```

### `init` execution flow

`init` performs these steps in order:

1. Resolve site name and read templates concurrently:

```ts
const [templates, siteName] = await Promise.all([
  readTemplates(),
  getSiteName(reqName, rootDir),
]);
const dest = path.resolve(rootDir, siteName);
```

2. Produce a `Source` with `getSource`:

```ts
const source = await getSource(reqTemplate, templates, cliOptions);
```

3. Materialize the source:

- Git source:

```ts
if (source.type === 'git') {
  if (!(await runGitCloneCommand(source, dest))) {
    logger.error`Cloning Git template failed!`;
    process.exit(1);
  }
  if (source.strategy === 'copy') {
    await fs.rm(path.join(dest, '.git'), {
      force: true,
      recursive: true,
    });
  }
}
```

- Built-in template source:

```ts
if (source.type === 'template') {
  await copyTemplate(source.template, dest, source.language);
}
```

- Local directory source:

```ts
await fs.cp(source.path, dest, {recursive: true});
```

4. Update `package.json`:

```ts
await updatePkg(path.join(dest, 'package.json'), {
  name: siteNameToPackageName(siteName),
  version: '0.0.0',
  private: true,
});
```

5. Rename template `gitignore` file to `.gitignore`:

```ts
if (
  !(await pathExists(path.join(dest, '.gitignore'))) &&
  (await pathExists(path.join(dest, 'gitignore')))
) {
  await fs.rename(path.join(dest, 'gitignore'), path.join(dest, '.gitignore'));
}
if (await pathExists(path.join(dest, 'gitignore'))) {
  await fs.rm(path.join(dest, 'gitignore'));
}
```

6. Select package manager and install deps:

```ts
const pkgManager = await getPackageManager(dest, cliOptions);
if (!cliOptions.skipInstall) {
  process.chdir(dest);
  if (!(await runPackageManagerInstallCommand(pkgManager))) {
    // log retry instructions
    process.exit(0);
  }
}
```

7. Print package manager help:

```ts
printPackageManagerHelp({pkgManager, cdpath});
```

`cdpath` is calculated as:

```ts
const cdpath = path.relative('.', dest);
```

---

## Source model

File: `packages/create-docusaurus/src/constants.ts`

The `Source` type is the internal representation of what to put in the destination directory:

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

A `Template` is:

```ts
export type Template = {
  name: string;
  path: string;
  tsVariantPath: string | undefined;
};
```

Supported Git clone strategies:

```ts
export const GitCloneStrategies = [
  'deep',
  'shallow',
  'copy',
  'custom',
] as const;

export type GitCloneStrategy = (typeof GitCloneStrategies)[number];
```

Supported package managers and lockfile names:

```ts
export const DefaultPackageManager = 'npm';

export const LockfileNames = {
  npm: 'package-lock.json',
  yarn: 'yarn.lock',
  pnpm: 'pnpm-lock.yaml',
  bun: 'bun.lockb',
};

export type PackageManager = keyof typeof LockfileNames;

export const PackageManagers = Object.keys(LockfileNames) as PackageManager[];
```

The order in `LockfileNames` is significant: `findPackageManagerFromLockFile` and `findPackageManagerFromUserAgent` iterate over `PackageManagers` in insertion order.

---

## Template discovery

File: `packages/create-docusaurus/src/index.ts`

Built-in templates are read from the `templates` directory relative to the source file:

```ts
const recommendedTemplate = 'classic';
const typeScriptTemplateSuffix = '-typescript';
const templatesDir = fileURLToPath(new URL('../templates', import.meta.url));
```

`readTemplates` filters the directory contents:

```ts
async function readTemplates(): Promise<Template[]> {
  const dirContents = await fs.readdir(templatesDir);
  const templates = await Promise.all(
    dirContents
      .filter(
        (d) =>
          !d.startsWith('.') &&
          !d.startsWith('README') &&
          !d.endsWith(typeScriptTemplateSuffix) &&
          d !== 'shared',
      )
      .map(async (name) => {
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
      }),
  );

  return templates.sort((a, b) => {
    if (a.name === recommendedTemplate) return -1;
    if (b.name === recommendedTemplate) return 1;
    return 0;
  });
}
```

This means:

- `shared` is not treated as a template.
- Any directory ending in `-typescript` is treated as a TypeScript variant, not a top-level template.
- The `classic` template is always sorted first.

---

## Source resolution

File: `packages/create-docusaurus/src/index.ts`

The top-level resolver is `getSource`:

```ts
async function getSource(
  reqTemplate: string | undefined,
  templates: Template[],
  cliOptions: CLIOptions,
): Promise<Source> {
  if (reqTemplate) {
    return getUserProvidedSource({reqTemplate, templates, cliOptions});
  }

  const template = await askTemplateChoice({templates, cliOptions});
  if (template === 'Git repository') {
    return askGitRepositorySource({cliOptions});
  }
  if (template === 'Local template') {
    return askLocalSource();
  }
  return createTemplateSource({template, cliOptions});
}
```

When the user passes a template argument, `getUserProvidedSource` classifies it:

```ts
async function getUserProvidedSource({
  reqTemplate,
  templates,
  cliOptions,
}: {
  reqTemplate: string;
  templates: Template[];
  cliOptions: CLIOptions;
}): Promise<Source> {
  if (isValidGitRepoUrl(reqTemplate)) {
    // Validate git strategy, return git source
    return {
      type: 'git',
      url: reqTemplate,
      strategy: cliOptions.gitStrategy ?? 'deep',
    };
  }
  if (await pathExists(path.resolve(reqTemplate))) {
    return {
      type: 'local',
      path: path.resolve(reqTemplate),
    };
  }
  return getTemplateSource({
    templateName: reqTemplate,
    templates,
    cliOptions,
  });
}
```

A Git repository URL is recognized by the prefixes `https://` and `git@`:

```ts
function isValidGitRepoUrl(gitRepoUrl: string): boolean {
  return ['https://', 'git@'].some((item) => gitRepoUrl.startsWith(item));
}
```

### Built-in template source

`getTemplateSource` looks up the requested template name:

```ts
async function getTemplateSource({
  templateName,
  templates,
  cliOptions,
}: {
  templateName: string;
  templates: Template[];
  cliOptions: CLIOptions;
}): Promise<Source> {
  const template = templates.find((t) => t.name === templateName);
  if (!template) {
    logger.error('Invalid template.');
    process.exit(1);
  }
  return createTemplateSource({template, cliOptions});
}
```

`createTemplateSource` resolves the language and validates that a TypeScript variant exists when requested:

```ts
async function createTemplateSource({
  template,
  cliOptions,
}: {
  template: Template;
  cliOptions: CLIOptions;
}): Promise<Source> {
  const language = await getLanguage(cliOptions);
  if (language === 'typescript' && !template.tsVariantPath) {
    logger.error`Template name=${template.name} doesn't provide a TypeScript variant.`;
    process.exit(1);
  }
  return {
    type: 'template',
    template,
    language,
  };
}
```

### Language selection

`getLanguage` uses the following precedence:

1. `--typescript`
2. `--javascript`
3. Prompt the user

```ts
async function getLanguage(options: LanguagesOptions) {
  if (options.typescript) return 'typescript';
  if (options.javascript) return 'javascript';
  return askPreferredLanguage();
}
```

---

## Template copying

File: `packages/create-docusaurus/src/index.ts`

`copyTemplate` always copies the shared assets first and then overlays the selected template:

```ts
async function copyTemplate(
  template: Template,
  dest: string,
  language: 'javascript' | 'typescript',
): Promise<void> {
  await fs.cp(path.join(templatesDir, 'shared'), dest, {
    recursive: true,
  });

  const sourcePath =
    language === 'typescript' ? template.tsVariantPath! : template.path;

  await fs.cp(sourcePath, dest, {
    recursive: true,
    filter: async (filePath) => !(await fs.lstat(filePath)).isSymbolicLink(),
  });
}
```

Important details:

- `templates/shared` is copied first.
- The language-specific template path is copied second.
- Symbolic links are filtered out during copy. The source comments note that symlinks do not exist in published npm packages anymore; this prevents errors during local testing.

---

## Command execution

File: `packages/create-docusaurus/src/commands.ts`

Commands are executed through `cross-spawn` instead of `node:child_process` for Windows compatibility. The module explains:

> We use cross-spawn instead of spawn because of Windows compatibility issues.
> For example, "yarn" doesn't work on Windows, it requires "yarn.cmd"

The core command runner:

```ts
async function runCommand(
  command: string,
  args: string[] = [],
  options: SpawnOptions = {},
): Promise<number> {
  const [realCommand, ...baseArgs] = command.split(' ');
  const allArgs = [...baseArgs, ...args];
  if (!realCommand) {
    throw new Error(`Invalid command: ${command}`);
  }

  return new Promise<number>((resolve, reject) => {
    const p = crossSpawn(realCommand, allArgs, {stdio: 'ignore', ...options});
    p.on('error', reject);
    p.on('close', (exitCode) =>
      exitCode !== null
        ? resolve(exitCode)
        : reject(new Error(`No exit code for command ${command}`)),
    );
  });
}
```

This is similar to `execa.command`, splitting the command string into the command and base args before combining with explicit args.

### Package manager availability

```ts
async function hasPackageManager(
  packageManager: PackageManager,
): Promise<boolean> {
  return (await runCommand(packageManager, ['--version'])) === 0;
}

export async function getAvailablePackageManagers(): Promise<PackageManager[]> {
  const list = await Promise.all(
    PackageManagers.map(async (name) => {
      return (await hasPackageManager(name)) ? name : null;
    }),
  );
  return list.filter((item) => item !== null);
}
```

### Package manager install command

```ts
export async function runPackageManagerInstallCommand(
  pkgManager: PackageManager,
): Promise<boolean> {
  const installCommand =
    pkgManager === 'yarn'
      ? 'yarn'
      : pkgManager === 'bun'
        ? 'bun install'
        : `${pkgManager} install --color always`;

  return (
    (await runCommand(installCommand, [], {
      env: {
        ...process.env,
        ...(supportsColor.stdout ? {FORCE_COLOR: '1'} : {}),
      },
    })) === 0
  );
}
```

The effective install command by package manager:

| Package manager | Command |
| --- | --- |
| `npm` | `npm install --color always` |
| `pnpm` | `pnpm install --color always` |
| `yarn` | `yarn` |
| `bun` | `bun install` |

Color output is forced through `FORCE_COLOR=1` when `supports-color` detects a color-capable stdout.

### Git clone command

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

export async function runGitCloneCommand(
  source: Source & {type: 'git'},
  dest: string,
): Promise<boolean> {
  const gitCommand = await getGitCloneCommand(source.strategy);
  return (await runCommand(gitCommand, [source.url, dest])) === 0;
}
```

Git strategy mapping:

| Strategy | Git command |
| --- | --- |
| `deep` | `git clone` |
| `shallow` | `git clone --recursive --depth 1` |
| `copy` | `git clone --recursive --depth 1` |
| `custom` | User-provided command via prompt |

`copy` differs from `shallow` only after cloning: `init` removes the `.git` directory.

---

## Package manager resolution

File: `packages/create-docusaurus/src/index.ts`

`getPackageManager` decides which package manager to use:

```ts
async function getPackageManager(
  dest: string,
  {packageManager, skipInstall}: CLIOptions,
): Promise<PackageManager> {
  if (packageManager && !PackageManagers.includes(packageManager)) {
    throw new Error(
      `Invalid package manager choice ${packageManager}. Must be one of ${PackageManagers.join(', ')}`,
    );
  }

  return (
    (await findPackageManagerFromLockFile(dest)) ??
    packageManager ??
    (await findPackageManagerFromLockFile('.')) ??
    findPackageManagerFromUserAgent() ??
    (skipInstall ? DefaultPackageManager : await askForPackageManagerChoice())
  );
}
```

Resolution order:

1. Lockfile in destination directory.
2. `--package-manager` CLI option.
3. Lockfile in the current working directory.
4. `npm_config_user_agent` environment variable.
5. `npm` if `--skip-install`.
6. Interactive package manager prompt.

### Lockfile detection

```ts
async function findPackageManagerFromLockFile(
  rootDir: string,
): Promise<PackageManager | undefined> {
  for (const packageManager of PackageManagers) {
    const lockFilePath = path.join(rootDir, LockfileNames[packageManager]);
    if (await pathExists(lockFilePath)) {
      return packageManager;
    }
  }
  return undefined;
}
```

### User agent detection

```ts
function findPackageManagerFromUserAgent(): PackageManager | undefined {
  return PackageManagers.find((packageManager) =>
    process.env.npm_config_user_agent?.startsWith(packageManager),
  );
}
```

### Interactive package manager selection

```ts
async function askForPackageManagerChoice(): Promise<PackageManager> {
  const packageManagers = await getAvailablePackageManagers();
  if (packageManagers.length === 0) {
    logger.warn`No package maintainer available? Trying with name=${DefaultPackageManager}`;
    return DefaultPackageManager;
  }
  if (packageManagers.length === 1) {
    return packageManagers[0]!;
  }

  const choices = packageManagers.map((p) => ({title: p, value: p}));
  return (
    (
      await prompts(
        {
          type: 'select',
          name: 'packageManager',
          message: 'Select a package manager...',
          choices,
        },
        {
          onCancel() {
            logger.info`Falling back to name=${DefaultPackageManager}`;
          },
        },
      )
    ).packageManager ?? DefaultPackageManager
  );
}
```

If one package manager is installed, the prompt is skipped and that package manager is used.

---

## Prompts

File: `packages/create-docusaurus/src/prompts.ts`

### Language prompt

```ts
export async function askPreferredLanguage(): Promise<
  'javascript' | 'typescript'
> {
  const {language} = await prompts({
    type: 'select',
    name: 'language',
    message: 'Which language do you want to use?',
    choices: [
      {title: logger.bold('JavaScript'), value: 'javascript'},
      {title: logger.bold('TypeScript'), value: 'typescript'},
    ],
  });
  if (!language) {
    process.exit(0);
  }
  return language;
}
```

### Custom Git clone command prompt

```ts
export async function askForCustomGitCloneCommand(): Promise<string> {
  const {command} = await prompts(
    {
      type: 'text',
      name: 'command',
      message:
        'Write your own git clone command. The repository URL and destination directory will be supplied. E.g. "git clone --depth 10"',
    },
    {
      onCancel() {
        logger.info`Falling back to code=${'git clone'}`;
      },
    },
  );
  return command ?? 'git clone';
}
```

---

## Utility functions

File: `packages/create-docusaurus/src/utils.ts`

### Site name to package name

`siteNameToPackageName` implements a lightweight kebab-case conversion. The package intentionally avoids a lodash dependency in this CLI package:

```ts
export function siteNameToPackageName(siteName: string): string {
  const match = siteName.match(
    /[A-Z]{2,}(?=[A-Z][a-z]+\d*|\b|_)|[A-Z]?[a-z]+\d*|[A-Z]|\d+/g,
  );
  if (match) {
    return match.map((x) => x.toLowerCase()).join('-');
  }
  return siteName;
}
```

### `package.json` update

```ts
export async function updatePkg(
  pkgPath: string,
  obj: {[key: string]: unknown},
): Promise<void> {
  const pkg = JSON.parse(await fs.readFile(pkgPath, 'utf8')) as {
    [key: string]: unknown;
  };
  const newPkg = Object.assign(pkg, obj);

  await fs.mkdir(path.dirname(pkgPath), {recursive: true});
  await fs.writeFile(pkgPath, `${JSON.stringify(newPkg, null, 2)}\n`);
}
```

The generated package always receives:

```ts
{
  name: siteNameToPackageName(siteName),
  version: '0.0.0',
  private: true,
}
```

### Path existence check

The package avoids `fs-extra` by implementing its own `pathExists`:

```ts
export async function pathExists(filePath: string): Promise<boolean> {
  return fs
    .access(filePath, fs.constants.F_OK)
    .then(() => true)
    .catch(() => false);
}
```

### Package manager help output

`printPackageManagerHelp` prints the final instructions:

```ts
export function printPackageManagerHelp({
  pkgManager,
  cdpath,
}: {
  pkgManager: PackageManager;
  cdpath: string;
}) {
  const useNpm = pkgManager === 'npm';
  const useBun = pkgManager === 'bun';
  const run = useNpm || useBun ? 'run ' : '';
  // logger output...
}
```

The `run` prefix is included only for `npm` and `bun`. For `yarn` and `pnpm`, the generated commands use the shorter form, for example `yarn build` instead of `yarn run build`.

---

## Built-in template files

The classic template contains configuration that is directly copied into new sites.

### `docusaurus.config.js`

File: `packages/create-docusaurus/templates/classic/docusaurus.config.js`

The config includes:

- `url`, `baseUrl`, `organizationName`, and `projectName`
- `onBrokenLinks: 'throw'`
- `i18n` with `defaultLocale: 'en'`
- `presets` using `classic`
- `themeConfig` with `navbar`, `footer`, and Prism themes

The classic preset options enable docs and blog plugins:

```js
presets: [
  [
    'classic',
    {
      docs: {
        sidebarPath: './sidebars.js',
        editUrl:
          'https://github.com/facebook/docusaurus/tree/main/packages/create-docusaurus/templates/shared/',
      },