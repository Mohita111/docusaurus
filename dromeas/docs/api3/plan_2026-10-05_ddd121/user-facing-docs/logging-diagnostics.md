# Logging and diagnostics

Logging & Diagnostics provides consistent, color-coded console output for all Docusaurus commands, along with optional performance metrics to help you understand what your site is doing and troubleshoot issues quickly.

## What it is and who it's for

Logging & Diagnostics is a built-in system that captures and displays real-time feedback from Docusaurus whenever you run commands like`docusaurus build` or`docusaurus serve`. It's designed for developers who want clear visibility into their build process, warnings about configuration issues, and detailed performance measurements.

The feature includes two complementary systems: a standard logger that handles all routine messages and warnings, and a performance logger that measures build times and memory usage when enabled.

## What you can do with it

**View clear, color-coded messages**

All console output is formatted with consistent prefixes and colors so you can quickly identify what type of message you're seeing:
- **Success messages** appear in green with a`[SUCCESS]` prefix
- **Information messages** appear in cyan with an`[INFO]` prefix
- **Warnings** appear in yellow with a`[WARNING]` prefix
- **Errors** appear in red with an`[ERROR]` prefix

File paths, URLs, code snippets, and other important values are highlighted with distinct styling so they stand out in the output.

**Receive configuration warnings**

During your build or development process, Docusaurus analyzes your configuration and reports issues it finds. You can control how these warnings are handled—they can be logged as information, elevated to warnings, or even halt the build entirely, depending on your preference.

**Measure build performance and memory**

When you enable performance logging, Docusaurus tracks how long each major operation takes and monitors memory usage throughout the build. This helps you identify bottlenecks and understand resource consumption.

## Key capabilities

### Standard logging

The logger provides several message types:

- **Info messages** tell you what's happening (e.g., "Starting build...")
- **Warnings** alert you to potential issues that don't stop the process (e.g., "Deprecated option detected")
- **Errors** report problems that cause the process to stop
- **Success messages** confirm that an operation completed without issues

Message text can include styled components for paths, URLs, code, numbers, and names, making it easy to scan complex output.

### Performance monitoring

When enabled via the`DOCUSAURUS_PERF_LOGGER` environment variable, the performance logger measures:

- **Operation duration**: How long each task took, displayed in milliseconds or seconds
- **Memory usage**: Heap memory before and after each operation, and total available heap
- **Parent context**: Nested operations show their parent's context (e.g., "Parent1 > Parent2 > child task")
- **Status indicators**: Shows whether an operation succeeded or failed

The performance logger color-codes duration thresholds:
- Operations under 5 milliseconds are not logged (below the minimum threshold)
- Operations between 5 and 100 milliseconds appear in green
- Operations between 100 milliseconds and 1 second appear in yellow
- Operations over 1 second appear in red

Memory is displayed in megabytes and shows both heap usage and total available heap memory.

## How to use it

### Read the output

As your build runs, watch the console for colored messages. The prefix tells you the message type, and the styled text within each message highlights important details like file paths and values. If performance logging is enabled, you'll also see`[PERF]` entries showing how long operations took and how much memory was used.

### Control warning severity

Docusaurus allows you to configure how warnings are reported through the reporting severity setting, which can be set to:
-`ignore`: warnings are not displayed
-`log`: warnings appear as info-level messages
-`warn`: warnings appear with yellow highlighting and a warning prefix
-`throw`: warnings stop the process and raise an error

## Related guides

- How to debug errors with detailed context
- How to measure build performance and memory usage
- How to receive warnings for configuration issues
- How to view build progress and status messages
