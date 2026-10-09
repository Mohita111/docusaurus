# How to extract translation messages from source code

Learn how to generate and update translation files for the text you have marked as translatable in your project.

## Prerequisites

- You have a Docusaurus project.
- Your source code already uses the Docusaurus **Translate** component or **translate** function to mark text for translation.
- You can run commands from the root folder of your project.

## Steps

1. Open a terminal and go to the root folder of your Docusaurus project.

2. Run the following command:

   ```bash
   docusaurus write-translations
   ```

3. Docusaurus scans your source code for two kinds of marked text:

   - Text inside a **Translate** component.
   - Text passed to the **translate** function.

4. The command reads each marked string and writes the extracted messages to the translation files in your project.

5. Review the terminal output for any warnings about messages that could not be extracted.

## Expected result

- Translation files are created or updated in the i18n area of your project.
- Each extracted message is included with its text and any optional description you provided.
- You can now provide translations for the extracted messages using the same files.

## Common issues

- If a message was not extracted, make sure the **Translate** component or **translate** function is imported from Docusaurus. The extraction only recognizes these two names.

- If you use a **Translate** component with no visible text, you must provide an **id**. For example, `<Translate id="welcome" />` is valid, but `<Translate />` is not.

- If you use the **translate** function, the first argument must be a static object. Dynamic values are not supported because they prevent automatic extraction.

- Text inside **Translate** and values passed to **translate** must be static strings. If you build a string by joining variables or calling functions, Docusaurus will warn you and will not extract that message.