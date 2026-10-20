# Logging and diagnostics

Logging & Diagnostics brings together all the console output, error reporting, code warnings, and test results you need to understand and troubleshoot a TanStack Start project from the terminal.

## What it is and who it's for

Logging & Diagnostics is the built-in reporting system that activates when you run any major project command, such as starting the development server, building the project, running tests, or checking code quality. It is designed for developers who want clear feedback about what the project is doing, where errors happen, and what needs attention.

The system combines several channels:

- **Command output** from the development server, production build, and preview commands
- **Automatic error logging** that is already wired into the project's configuration
- **Code warnings and errors** from ESLint when you run the lint command
- **Test results** from Vitest when you run test suites
- **Environment checks** for port, host, and strict-port settings

## What you can do with it

### Check code quality with ESLint

Run`npm run lint` to scan all project files. The output separates issues by severity:

- **Errors** identify problems you should fix. For example, importing the Next.js`server-only` module is rejected because this project uses the TanStack Start equivalent instead.
- **Warnings** point to issues that do not stop the project. For example, exporting something from a component that prevents React fast refresh is reported as a warning, with constant exports allowed.

Lint results include file names and a short explanation for each issue, so you can find and fix the source quickly.
