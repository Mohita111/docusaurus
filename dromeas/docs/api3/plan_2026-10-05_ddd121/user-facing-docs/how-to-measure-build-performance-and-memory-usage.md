# How to measure build performance and memory usage

When you build your Docusaurus site, you want to know how long it takes and how much memory it consumes. This guide shows you how to enable performance monitoring and interpret the results.

## Prerequisites

- Node.js installed on your system
- A Docusaurus project set up and ready to build
- Command-line access to run build commands with special flags

## Enable performance logging

Performance monitoring in Docusaurus is controlled by an environment variable. When you enable it, the build process reports detailed timing and memory information for each major operation.

1. Open your terminal or command prompt
2. Set the environment variable before running your build command

**On macOS or Linux:**

```bash
DOCUSAURUS_PERF_LOGGER=true npx docusaurus build
```

**On Windows (PowerShell):**

```powershell
$env:DOCUSAURUS_PERF_LOGGER='true'; npx docusaurus build
```

**On Windows (Command Prompt):**

```cmd
set DOCUSAURUS_PERF_LOGGER=true && npx docusaurus build
```

## Understand the performance output

When performance logging is enabled, you'll see lines in your CLI output starting with **[PERF]**. Each line shows:

- **Operation name** – what Docusaurus is doing (for example, "Loading plugins" or "Bundling assets")
- **Duration** – how long the operation took, color-coded:
  - **Green** – fast (under 100 milliseconds)
  - **Yellow** – moderate (100 to 1000 milliseconds)
  - **Red** – slow (over 1 second)
- **Memory information** – heap memory before and after the operation, plus total heap available

Example output line:

```
[PERF] Generating site bundle - 2345.67 ms - (Heap 145mb -> 342mb / Total 1024mb)
```

This tells you that generating the bundle took 2.3 seconds, and heap memory increased from 145 MB to 342 MB during that operation.

## Enable garbage collection information

For more detailed memory insights, run the build with Node.js's garbage collection flag exposed:

```bash
DOCUSAURUS_PERF_LOGGER=true node --expose-gc ./node_modules/.bin/docusaurus build
```

With this flag, the performance logger explicitly triggers garbage collection before measuring memory. This gives you cleaner memory readings that reflect actual usage rather than residual allocations.

## Identify performance bottlenecks

Performance logs appear in hierarchical order, showing parent operations followed by their child operations. Look for:

1. **Operations with red duration values** – these are your slowest steps. Focus optimization efforts here first
2. **Large heap jumps** – when memory increases significantly during an operation, it may indicate inefficient processing
3. **Nested slow operations** – if a parent operation is slow, check which child operations contribute most to that time

## Expected results

A typical build output shows multiple **[PERF]** entries as Docusaurus:

- Loads configuration
- Processes plugins
- Generates site pages
- Bundles assets
- Optimizes output

You should see memory usage increase and decrease throughout the build as different operations run.

## Common issues

**No performance output appears** – Verify that you set the environment variable correctly. The value must be exactly`'true'` (lowercase). Check that you're running the command in the same terminal where you set the variable.

**Memory keeps increasing** – This is often normal during a build, especially with large sites. The **[PERF]** logs show memory before and after each operation, so you can see if individual operations are holding onto memory unexpectedly.

**Operations show no timing** – Very fast operations (under 5 milliseconds) don't appear in performance logs to reduce output noise. This is expected behavior.

**Heap memory is very high** – Consider increasing your Node.js heap size if you see "out of memory" errors. You can do this with the`--max-old-space-size` flag:

```bash
DOCUSAURUS_PERF_LOGGER=true node --max-old-space-size=4096 ./node_modules/.bin/docusaurus build
```

## Next steps

After identifying slow operations, you can optimize your site. Common improvements include enabling faster build modes and using alternative minifiers for better build performance.

For more information about understanding build output and warnings, see how to view build progress and status messages.
