# be plugin for claude code and codex

This repo defines the be plugin for Claude Code and Codex. You are a
coding assistant helping the user to develop the be plugin.

## Two layers

This repo has two layers, easy to confuse:

- **The be plugin**: the code under `plugins/be`, which anyone who
  installs the plugin gets.
- **Developer support**: everything else, such as this file, the lint
  config, and `scripts/`. It supports plugin development and is not part
  of the installed plugin.

## Skill references

The prose in this repo references a skill one way throughout:

- Write a skill reference as `be:foo`, with no leading slash. Both hosts
  resolve that identifier, so an agent can act on it as written.
- Keep the leading slash, `/be:foo`, where the reference stands for what
  a person types: the README, a frontmatter `description:` field, and a
  line the agent shows the user.
- Drop the backticks in a frontmatter `description:` field, a markdown
  heading, or a fenced command block.
- Write a skill file's title heading as `be:foo`, so it names the skill
  a reader came looking for. `scripts/validate_skill.py` checks this on
  commit.

This repo is mostly plugin metadata and skills. There is no test suite.
Run the pre-commit hooks to check formatting. See the Linting section.

## Linting

The repo uses [`pre-commit`](https://pre-commit.com/) for lightweight
checks:

- trailing whitespace, end-of-file newlines, and JSON syntax
- Markdown style (`markdownlint-cli2`, with tuned rules in
  `.markdownlint.json`)
- Markdown formatting (`prettier`, which reflows prose to a 72-column
  width, so lines never need wrapping by hand)
- Markdown link validation (`remark-validate-links`, which checks
  within-file and cross-file anchor links resolve to real headings)
- invisible characters such as non-breaking spaces, zero-width marks,
  and bidi controls (`scripts/check_invisible_chars.py`)
- `claude plugin validate` on the plugin and marketplace manifests
- YAML frontmatter and title heading validation on skill files
- the documentation index (`uncoded sync`, which regenerates
  `.uncoded/docs.yaml` and the `uncoded-doc-navigation` skill)

At the start of each session, pull the latest `main` and install the
hooks:

```bash
git pull origin main
uvx pre-commit install
```

Run all hooks once: `uvx pre-commit run --all-files`. The same hooks run
in CI on every push and pull request (see `.github/workflows/lint.yml`).
The `claude plugin validate` hook requires the Claude Code CLI on
`PATH`. CI installs it via `npm`. The `uncoded` hook requires `uvx` on
`PATH`. CI provides it via `astral-sh/setup-uv`.

## Release protocol

There is no release process. The plugin is installed directly from this
GitHub repo's main branch.

When opening a PR, include a version bump in
`plugins/be/.codex-plugin/plugin.json`, so every change merged to main
is versioned.

The manifest sits under `.codex-plugin` because Codex drops symlinks
when it installs a plugin, and Claude Code follows them. So
`plugins/be/.claude-plugin/plugin.json` is a symlink to it, and both
hosts read one file.

`claude plugin validate` reads that file too, and warns that it ignores
the `interface` block. Codex needs the block, so leave it in place.

Which part to bump:

- **Major**: a structural or breaking change to existing behaviour.
- **Minor**: an additive, non-breaking change.
- **Micro**: a bug fix.

A change that touches only developer support needs no bump, because it
is not part of the installed plugin.

Keep the PR description short and current: say what the PR does and why,
and leave line-by-line detail to the diff.

## Before you start

Load the `uncoded-doc-navigation` skill before searching, reading or
editing any Markdown files in this codebase.
