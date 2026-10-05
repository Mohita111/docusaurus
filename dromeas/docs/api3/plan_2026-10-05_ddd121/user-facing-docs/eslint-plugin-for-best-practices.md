# ESLint plugin for best practices

The ESLint plugin for Docusaurus helps your documentation team catch mistakes early and enforce consistent practices across your site. It works alongside ESLint to validate your documentation code, configuration, and content structure—catching issues that could affect accessibility, performance, or user experience.

## What this feature is for

This plugin is designed for development teams building Docusaurus sites who want to:

- Enforce Docusaurus-specific coding conventions automatically
- Catch common mistakes before they reach production
- Ensure your team follows accessibility and performance best practices
- Integrate documentation quality checks into your existing ESLint workflow
- Maintain consistency across multiple documentation projects or contributors

The plugin is especially valuable if you have a team of contributors, want to prevent anti-patterns in your documentation structure, or need to meet accessibility and performance standards.

## What the plugin does

The ESLint plugin validates your documentation project against Docusaurus best practices. It analyzes your code and configuration to identify patterns that could cause problems, then reports them as ESLint messages you can fix before building your site.

### Key capabilities

**Detects documentation anti-patterns**, The plugin recognizes common mistakes in how documentation is structured and organized. It catches issues that might not cause immediate build failures but could make your documentation harder to maintain or less effective for users.

**Validates MDX structure**, When you write documentation in MDX format, the plugin ensures your content follows proper structure and conventions. This helps prevent formatting issues and ensures your interactive components work as intended.

**Enforces configuration best practices**, The plugin checks your Docusaurus configuration and plugin settings to ensure they follow recommended patterns. This catches configuration mistakes that might cause subtle bugs or performance degradation.

**Encourages accessibility standards**, The plugin identifies content and code patterns that could create accessibility barriers. It prompts you to fix issues that might affect users with assistive technologies, helping you build more inclusive documentation.

**Warns about performance concerns**, The plugin flags patterns that could negatively affect your site's performance, such as inefficient image handling, oversized bundles, or problematic component usage. These warnings help you optimize your documentation before performance issues impact users.

## How to use the plugin

The plugin integrates directly into your ESLint setup. Once you've installed and configured it in your ESLint configuration file, it automatically checks your documentation files whenever ESLint runs—whether through your editor, pre-commit hooks, or CI/CD pipeline.

### Installing the plugin

The plugin requires ESLint version 8.57.0 or later and Node.js version 24.21 or later. Install it alongside your project dependencies using your package manager, then add it to your ESLint configuration.

### Configuring rules

Once installed, you control which rules the plugin enforces by adding them to your ESLint configuration. You can:

- Enable or disable individual rules based on your team's needs
- Set each rule to warn or error, letting you choose whether violations block your build
- Exclude specific files or directories from linting if needed

This flexibility means you can start with strict enforcement and gradually adopt rules, or customize the plugin to match your team's preferences and existing practices.

## How it fits with your workflow

The plugin works silently in the background as part of your normal ESLint setup. When you save a file in your editor, run ESLint in your CI/CD pipeline, or build your documentation, the plugin checks your code against its rules. If it finds violations, it reports them as ESLint messages with helpful descriptions, just like any other ESLint rule.

Because it integrates with standard ESLint tools and workflows, your team can use familiar commands and processes—you're just adding Docusaurus-specific intelligence to your existing linting setup.

## What the plugin doesn't do

The plugin focuses on code and configuration validation, not content quality. While it catches structural mistakes and anti-patterns, it won't evaluate whether your documentation is clear, complete, or helpful to users. That's a job for editorial review and user feedback.

The plugin also requires a working ESLint setup. If you're not already using ESLint, you'll need to set it up first before you can use this plugin.

## Benefits at a glance

By catching Docusaurus-specific mistakes automatically, the plugin helps your team:

- **Ship higher-quality documentation**, Problems are caught in development, not when users visit your site
- **Reduce back-and-forth reviews**, Automated checks handle consistency so your team can focus on content and design
- **Improve accessibility and performance without expertise**, The plugin guides you toward best practices, even if accessibility or performance isn't your specialty
- **Scale across teams**, New contributors automatically follow the same standards as experienced team members

## Related documentation

The plugin enhances your development workflow when combined with other Docusaurus features and best practices for managing your documentation project.
