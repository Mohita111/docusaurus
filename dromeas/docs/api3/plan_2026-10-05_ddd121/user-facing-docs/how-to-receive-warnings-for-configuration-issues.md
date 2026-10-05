# How to receive warnings for configuration issues

When you run Docusaurus commands, the build process reports configuration issues through your command-line interface (CLI). Docusaurus uses a logging system that displays warnings about potential problems with your setup, helping you catch issues before they affect your documentation site.

## Goal

Receive and understand warning messages during the build process so you can identify and fix configuration issues in your Docusaurus project.

## Prerequisites

- Docusaurus is installed on your system
- You have access to your project's command line
- You've initialized a Docusaurus project with a`docusaurus.config.js` file

## How configuration warnings are displayed

Docusaurus displays warnings with a **[WARNING]** prefix in yellow text in your CLI output. These warnings appear when the build detects configuration problems that don't prevent the build from completing, but could cause issues with your documentation site.

The warning system supports different severity levels that control how issues are reported:

- **ignore**: No message is shown (the issue is skipped silently)
- **log**: Displays as an informational message with a **[INFO]** prefix in cyan text
- **warn**: Displays as a warning with a **[WARNING]** prefix in yellow text (the default for configuration issues)
- **throw**: Stops the build immediately and displays an error with a **[ERROR]** prefix in red text

## Steps to see configuration warnings

1. **Open your terminal** and navigate to your Docusaurus project directory.

2. **Run the build or development server command**:
   - For a full production build, run:
```
     docusaurus build
     ```
   - For a development server with live reloading, run:
```
     docusaurus start
     ```

3. **Watch the CLI output** as Docusaurus processes your configuration. Any configuration issues will appear with the **[WARNING]** prefix in yellow text.

4. **Read the warning message carefully**. It describes what configuration issue was detected and where it occurs (such as a file path or configuration option name).

5. **Note the specific details** provided in the warning, such as:
   - File paths shown in cyan underlined text (example:`"path/to/file"`)
   - Configuration option names shown in bold blue text
   - Code snippets shown in cyan text with backticks

6. **Fix the configuration issue** based on the warning message by updating your`docusaurus.config.js` or relevant configuration files.

7. **Re-run the build or development command** to verify the warning has been resolved.

## Expected result

When your configuration is correct:
- Build completes successfully with no **[WARNING]** messages
- CLI output shows **[SUCCESS]** messages (in green text) and **[INFO]** messages (in cyan text)
- Your documentation site builds without errors

When configuration issues exist:
- **[WARNING]** messages (in yellow text) appear describing the specific issue
- Build continues to completion despite the warnings
- Your site may have unexpected behavior

## Understanding warning messages

Warning messages use a consistent format:

```
[WARNING] Description of the issue: path="path/to/file" or option=`configOption`
```

The message includes formatted elements to help you locate the problem:
- **File paths** appear in cyan underlined text
- **Code** and configuration options appear in backticks with cyan styling
- **Configuration values** appear with appropriate emphasis

## Common configuration warnings

Configuration warnings typically appear for issues such as:
- Missing or invalid plugin configurations
- Incorrect documentation paths or sidebar definitions
- Missing required metadata in front matter
- Conflicting configuration options
- Deprecated configuration patterns

## Troubleshooting warnings

**Warning doesn't go away after fixing the issue:**
- Clear your build cache by deleting the`.docusaurus` folder in your project root
- Run the build command again

**Can't understand what the warning is telling you:**
- Read the warning message twice—the first line usually identifies the exact issue
- Check the Logging & Diagnostics documentation for more details about specific warning types

**Want to treat warnings as errors:**
- Some projects prefer to fail the build when warnings occur
- This prevents configuration issues from reaching production
- Check your build configuration to see if you can change the severity level from`warn` to`throw`

## Related topics

- View build progress and status messages
- Debug errors with detailed context
- Logging & Diagnostics feature
