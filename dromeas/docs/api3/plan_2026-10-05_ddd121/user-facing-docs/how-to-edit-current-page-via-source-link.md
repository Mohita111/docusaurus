# How to edit the current page using the edit this page link

This guide explains how a content author can update a documentation page directly from the live site by opening the page source through the **Edit this page** link.

## Prerequisites

- You have the page open in your web browser
- You have a source editing account, such as GitHub or GitLab, that can edit the project where the documentation is stored
- You have permission to make changes to that project

## Steps

1. Open the documentation page you want to update
2. Scroll to the bottom of the page
3. Select **Edit this page**
4. If prompted, sign in to your source editing account
5. Make your changes in the editor that opens
6. Select the editor's commit or save action to submit your update

## Expected result

Your browser opens the source version of the page from step 1. After you save, your changes are submitted to the documentation project. When the site is refreshed or republished, the updated content appears on the live page.

## How the edit link is configured

The **Edit this page** link is controlled by the site administrator through the`editUrl` setting. This setting can be a plain URL (like`https://github.com/myorg/mydocs/edit/main/`) or a custom function that generates different URLs based on the page's version, language, and file location.

When`editUrl` is configured, the plugin automatically appends the relative path to your document file, creating the full link to edit that specific page.

## Common issues

**The Edit this page link is missing.** The link only appears when the site owner has enabled it in their configuration. Ask your site administrator if you need this feature enabled.

**The link opens a different version of the page than expected.** The site may be configured with`editCurrentVersion: true`, which means the edit link always targets the current version's documentation files instead of the specific version you were reading.

**The editor opens in read-only mode.** You may not have permission to edit the source directly. Request edit access from the project owner.

**The link opens an unexpected editing page.** The page author may have set a custom editing address for that specific page using the`custom_edit_url` front matter field.

**Your changes don't appear immediately.** Many documentation sites publish updates after a review or on a schedule, so allow time for the change to go live.
