# How to debug errors with detailed context

When you encounter errors during your Docusaurus build or development, you can enable detailed performance and memory diagnostics to understand what's happening and pinpoint the source of problems.

## Goal

Capture detailed error context including performance metrics, memory usage, and operation hierarchies to help you identify and fix build issues faster.

## Prerequisites

- You're running a Docusaurus CLI command (like`docusaurus build` or`docusaurus start`)
- Node.js is installed with the`--expose-gc` flag available (recommended for accurate memory reporting)
- You have access to your terminal or command line interface

## Enable performance debugging

Follow these numbered steps to activate detailed error context logging:

1. **Open your terminal or command prompt** where you run Docusaurus commands.

2. **Set the debug environment variable** before running your build command:

   On macOS or Linux:
```
   export DOCUSAURUS_PERF_LOGGER=true
   ```

   On Windows (Command Prompt):
```
   set DOCUSAURUS_PERF_LOGGER=true
   ```

   On Windows (PowerShell):
```
   $env:DOCUSAURUS_PERF_LOGGER='true'
   ```

3. **Run your Docusaurus command** to trigger the build or development process:
```
   docusaurus build
   ```
   or
```
   docusaurus start
   ```

4. **Review the performance output** in your terminal. You'll see messages that look like:

```
   [PERF] Operation name - 123.45 ms - (Heap 256mb -> 280mb / Total 512mb)
   ```

## Understand the error context output

When performance debugging is enabled, each operation displays:

- **Operation name**: The task being executed (like building pages or processing content)
- **Duration**: How long the operation took in milliseconds (ms) or seconds
- **Heap usage**: Memory used before → after the operation, plus total heap allocated
- **Color coding**:
  - Green text: Operations completed in under 100 ms (healthy)
  - Yellow text: Operations took 100–1000 ms (moderate)
  - Red text: Operations exceeded 1000 ms or failed (slow or errored)

### Example output hierarchy

The performance logger shows nested operations with a hierarchical structure:

```
[PERF] Build site - 2500.00 ms - (Heap 256mb -> 340mb / Total 512mb)
[PERF] Build site > Compile content - 1800.00 ms - (Heap 280mb -> 310mb / Total 512mb)
[PERF] Build site > Compile content > Process MDX - 1200.50 ms - (Heap 290mb -> 300mb / Total 512mb)
[PERF] Build site > Optimize assets - 700.00 ms - (Heap 310mb -> 340mb / Total 512mb)
```

This hierarchy (shown as`Parent > Child > Grandchild`) helps you trace where time is being spent in nested operations.

## Capture detailed memory snapshots

For advanced debugging, combine performance logging with garbage collection exposure:

1. **Run your build with garbage collection enabled**:

```
   node --expose-gc node_modules/.bin/docusaurus build
   ```

2. **Set the performance logger flag** in the same command:

```
   DOCUSAURUS_PERF_LOGGER=true node --expose-gc node_modules/.bin/docusaurus build
   ```

3. **Watch for memory growth patterns** in the heap usage output. Look for:
   - Steady memory increases that don't recover (potential memory leak)
   - Large jumps in heap usage during specific operations
   - Heap total growing beyond expected limits

## Expected results

When you enable error context debugging, you'll see:

- A **[PERF]** prefix on every timed operation in your CLI output
- **Operation names with parent hierarchy** showing the call stack (e.g.,`Build site > Compile content > Process markdown`)
- **Duration times** in milliseconds or seconds, color-coded by speed
- **Memory snapshots** showing heap usage before and after each operation
- **Error markers** (red [KO] status) when operations fail, with the error preserved in the thrown exception

This makes it easy to spot which operations are slow or consuming excessive memory, and at what nesting level the problem occurs.

## Common issues and solutions

**No performance output appears**

The environment variable may not be set correctly. Verify it's set before running the command:
- On macOS/Linux, type`echo $DOCUSAURUS_PERF_LOGGER` to confirm
- On Windows, type`echo %DOCUSAURUS_PERF_LOGGER%` to confirm
- The value must be exactly`true` (case-sensitive on some systems)

**Memory numbers seem inaccurate**

You didn't run Node.js with the`--expose-gc` flag. Garbage collection won't run automatically, so memory snapshots reflect usage with accumulated garbage. Use`node --expose-gc` to force collection before each measurement.

**Performance logs are truncated or missing for quick operations**

Operations faster than 5 milliseconds don't log anything (below the threshold). This is by design to reduce noise. Only operations taking 5 ms or longer appear in the output.

**Heap numbers keep growing even with garbage collection**

This may indicate a genuine memory leak in your code or content. Run the build multiple times and compare heap growth across runs. If it's consistent, file an issue with the specific operations shown in the performance log.

---

## Related

How to measure build performance and memory usage

How to view build progress and status messages

How to receive warnings for configuration issues
