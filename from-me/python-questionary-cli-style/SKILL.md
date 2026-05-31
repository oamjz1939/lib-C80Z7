---
name: python-questionary-cli-style
description: Use when designing or implementing a Python CLI with questionary, especially to enforce wizard-style interaction, prompt layout, title blocks, and structured terminal output.
---

# Python Questionary CLI Visual Style

## Purpose

Use this skill when designing or implementing a Python CLI with `questionary`.

Scope: visual layout, interaction presentation, and output style only. Do not use this skill to decide business logic, packaging, tests, or command architecture.

## Core Rules

Always prefer a wizard-style interactive CLI.

Before implementation, preview the intended visible CLI experience to the user: title, steps, prompts, interaction flow, and final visible result.

Every CLI must start with this title-block style:

```text
┌  Configure Python Project
│
```

For each single-choice prompt, ask the user which behavior they want:

- `questionary.rawselect()`: numbered list; pressing a number immediately selects and returns.
- `questionary.select()`: arrow keys move; Enter confirms; may use `use_shortcuts=True`.

## Components

Use:

- `questionary.text()` for text input.
- `questionary.select()` for normal single choice.
- `questionary.rawselect()` for immediate numeric choice.
- `questionary.checkbox()` only when multiple choices are truly needed.
- `questionary.confirm()` for yes/no decisions.
- `questionary.press_any_key_to_continue()` only when a pause is truly useful.

## Prompt Style

Prompts should be short, clear, and action-oriented. Show defaults clearly when relevant.

```text
Project name: (my-app)
Select project template:
Choose deployment target:
Initialize git repository? (Y/n)
Overwrite existing files? (y/N)
```

For confirm prompts, uppercase marks the default.

## Visual Style

The CLI should feel modern, quiet, and clear.

Use:

- whitespace instead of dense separators
- two-space indentation for hierarchy
- `✔` for completed successful steps
- `Error` for failures
- progress bars for long tasks when useful
- aligned columns for structured fields
- section headers like `── Name ─────`
- emoji only sparingly, usually in a title or high-level status line

Avoid visual noise.

## Structured Output

```text
┌  Deploy Application
│

? Select environment:
❯ prod
  staging
  dev

? Run database migrations? (y/N)

🚢 Shipping update to 'prod' environment...

── Build Pipeline ────────────────────────────────────────

  ✔  Linting and formatting checked
  ✔  Compiling assets (production mode) [0.4s]
  ✔  Building production binary [1.2s]

── Deployment ────────────────────────────────────────────

  Uploaded  [████████████████████████████████] 100% (12/12 files)

  ✔  Files uploaded successfully
  ✔  Remote service restarted

```

## Forbidden

Do not use:

- ASCII art banners
- large decorative headers
- repeated `====` separators
- rainbow-colored output
- emoji on every line
- excessive logging
- dense walls of terminal text
- decorative noise without meaning