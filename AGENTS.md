# Documentation project instructions

## About this project

- This is the Engy documentation, published with [GitBook](https://www.gitbook.com) through Git Sync
- Each top-level directory is one GitBook space. It contains its own `.gitbook.yaml`, `README.md` (the space landing page), and `SUMMARY.md` (the table of contents)
- Pages are Markdown files with optional YAML frontmatter. The page title is the first `#` heading, and the frontmatter `description` is the page summary
- Every page must be listed in its space's `SUMMARY.md`, or GitBook does not show it
- Store images for a space in that space's `.gitbook/assets/` directory
- Brand assets are in `brand/`. Upload them in the GitBook site settings, because GitBook does not read them from the repository

## GitBook blocks

Use GitBook block syntax, not MDX components:

| Purpose | Syntax |
| --- | --- |
| Callout | `{% hint style="info" %}` ... `{% endhint %}`. Styles: `info`, `success`, `warning`, `danger` |
| Sequential steps | `{% stepper %}` with one `{% step %}` ... `{% endstep %}` per step, each starting with a `###` heading |
| Tabs | `{% tabs %}` with one `{% tab title="..." %}` ... `{% endtab %}` per tab |
| Titled code block | `{% code title="file.py" %}` around a fenced code block ... `{% endcode %}` |

## Style preferences

- Use active voice and second person ("you")
- Keep sentences concise — one idea per sentence
- Use sentence case for headings
- Bold for UI elements: Click **Settings**
- Code formatting for file names, commands, paths, and code references
