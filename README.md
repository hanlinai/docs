# Engy documentation

This repository contains the source of the Engy documentation. GitBook publishes it through Git Sync.

## Repository structure

The documentation is published as two GitBook sites:

| Directory | GitBook space | Site | Audience |
| --- | --- | --- | --- |
| `documentation/` | Documentation | User docs (`docs.engy.ai`) | Users who integrate with Engy |
| `brand/` | Not synced | Not published | Logo and icon for the GitBook site settings |

The provider site will contain the `providers/` and `sn53-network/` spaces when their first pages are written.

Each space directory contains:

- `.gitbook.yaml`: the Git Sync configuration for the space
- `README.md`: the landing page of the space
- `SUMMARY.md`: the table of contents of the space

## Publish changes

1. In GitBook, open the space and enable **Git Sync** for this repository.
2. Set the project directory to the space directory, such as `documentation/`.
3. Push changes to the default branch. GitBook syncs them automatically.

## Brand settings

Configure the following values in the GitBook site customization settings:

| Setting | Value |
| --- | --- |
| Logo | `brand/logo.svg` |
| Icon | `brand/icon.svg` |
| Primary color (light mode) | `#E8541C` |
| Primary color (dark mode) | `#FF6A38` |
