# CodeBlend-Plugins

A [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) with tools to enhance your development workflow.

## Plugins

| Plugin | Description |
|--------|-------------|
| [codeblend-commit](./codeblend-commit) | Analyzes staged git changes and creates intelligent commits by categorizing them into logical groups |

## Installation

1. Add the marketplace:

   ```
   /plugin marketplace add kefeiqian/CodeBlend-Plugins
   ```

2. Install a plugin:

   ```
   /plugin install codeblend-commit@codeblend-plugins --scope user
   ```

## Usage

Once installed, use the `/codeblend-commit` skill when you're ready to commit. It will analyze your staged changes, categorize them, and propose well-structured commits.
