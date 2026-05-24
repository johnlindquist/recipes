# Recipes: Shareable Pi Agent Harness Collection

## Objective

Create a repo in `~/dev/recipes` for collecting easy-to-install, single-purpose Pi agent harnesses that replace bespoke zsh wrapper functions such as `cx`, `kx`, and `zx`.

Each harness should be isolated, focused on one scenario, easy to customize, and easy to share. The repo should also provide a simple installer that lets users choose which harnesses to install, then makes them available on the user's `PATH` through commands, aliases, or shell integration.

## Background

One recurring challenge with Codex workflows is creating bespoke, isolated, single-purpose agent harnesses. Existing local wrappers such as `cx`, `kx`, and `zx` solve this with zsh functions that launch Codex with a narrow prompt, suppressed ambient context, and task-specific CLI access.

`pi.dev` may be a better substrate for this pattern because it can use an existing Codex subscription while making harness customization easier than hand-maintained shell functions.

This repo should become the home for those harnesses: a cookbook of agentic tools that can be installed individually and shared with others.

## Desired User Experience

A user should be able to:

1. Clone or reference the `recipes` repo.
2. Run a single installer command from the hosted repo.
3. See a list of available harnesses.
4. Pick one or more harnesses to install.
5. Get stable commands on their `PATH`, such as `cx`, `kx`, `zx`, or future command names.
6. Run those commands from any directory with predictable, isolated behavior.
7. Update or uninstall installed harnesses without hand-editing shell config.

The installer should make the installed state obvious and reversible.

## Harness Model

Each top-level recipe directory should represent one Pi agent harness.

Suggested shape:

```text
recipes/
  .goals/
    recipes.md
  recipes/
    cx/
      recipe.json
      README.md
      prompt.md
      harness.ts
    kx/
      recipe.json
      README.md
      prompt.md
      harness.ts
    zx/
      recipe.json
      README.md
      prompt.md
      harness.ts
  installer/
    install.ts
  bin/
    recipes
```

The exact language is undecided. TypeScript is likely the best first default because it is fast to iterate, portable for CLI wrappers, familiar for config-heavy harnesses, and easy to distribute with `bun`, `tsx`, or compiled single-file JavaScript. Rust may be useful later for a fast standalone installer or binary launcher, but it should not be chosen before confirming Pi's integration model.

## Recipe Manifest

Each harness should include a manifest with enough metadata for the installer and documentation generator.

Example fields:

```json
{
  "name": "kx",
  "description": "Pi harness specialized for Karabiner config tasks",
  "command": "kx",
  "entry": "harness.ts",
  "prompt": "prompt.md",
  "category": "config",
  "requires": ["pi", "node"],
  "tools": ["karabiner", "git"],
  "defaultRoot": "~/.config",
  "interactiveCommand": "kxi"
}
```

The manifest should support both one-shot and interactive variants where that maps cleanly to Pi.

## Initial Recipes

### `cx`

General bare Codex/Pi harness equivalent to the existing `cx` zsh wrapper.

Requirements:

- Suppress irrelevant ambient context.
- Use a narrow developer/system prompt.
- Support one-shot task execution.
- Provide an interactive variant if Pi supports it cleanly.

### `kx`

Karabiner configuration expert harness.

Requirements:

- Default working root should be `~/.config`.
- Focus on `kar-migration/config.ts` and related Karabiner config files.
- Support help/debug prompt modes if useful.
- Preserve the current zsh wrapper's specialization, but express it as a shareable Pi harness.

### `zx`

Zsh wrapper factory and shell-config expert harness.

Requirements:

- Default working root should be `~/.config`.
- Focus on creating, editing, and reviewing task-specific zsh functions.
- Help migrate existing shell wrappers into recipes.
- Be able to reason about installability and shell startup behavior.

## Installer Requirements

The installer should be runnable from a hosted repo URL. For example:

```sh
curl -fsSL https://raw.githubusercontent.com/<owner>/recipes/main/install.sh | sh
```

The exact hosted URL can be filled in once the repo remote exists.

The installer should:

- Detect required runtimes.
- Offer an interactive picker for available recipes.
- Support non-interactive install flags, such as `--recipe kx --recipe zx`.
- Install commands into a predictable user-owned location, such as `~/.local/bin` or `~/.recipes/bin`.
- Add shell integration only with explicit user consent.
- Support zsh first, with room for bash and fish.
- Avoid overwriting existing commands without confirmation.
- Record installed recipes and versions.
- Provide `recipes list`, `recipes install`, `recipes update`, and `recipes uninstall` commands.

## Installation Strategy

Prefer installing small launcher scripts on the user's `PATH` rather than mutating large shell config blocks.

Possible layout:

```text
~/.recipes/
  recipes/
    kx/
    zx/
  bin/
    kx
    kxi
    zx
    zxi
  state.json
```

Shell config should only need a minimal path line:

```sh
export PATH="$HOME/.recipes/bin:$PATH"
```

If aliases are required, keep them generated in a single managed file:

```text
~/.recipes/shell/aliases.zsh
```

Then source that file from zsh with a clearly marked block.

## Open Questions

- What is Pi's recommended file format for harnesses?
- Does Pi prefer TypeScript, JavaScript, Python, Rust, or a config-only harness model?
- Can Pi support both one-shot and interactive modes for the same recipe?
- How does Pi authenticate against the user's Codex subscription?
- What is the cleanest way to pass per-recipe prompts and CLI/tool constraints?
- Should recipes be installed as source files, compiled launchers, or generated Pi config?
- Should command names intentionally match existing wrappers (`cx`, `kx`, `zx`) or use a namespace such as `recipe-kx` to avoid collisions?

## Implementation Plan

1. Research Pi's current harness format and runtime expectations.
2. Choose the smallest viable recipe format.
3. Initialize this directory as a real repo with baseline `README.md`, `.gitignore`, and installer scaffold.
4. Implement the manifest schema.
5. Build one recipe end to end, preferably `kx`, because it has a specific target root and task domain.
6. Add `cx` and `zx` after the recipe contract is proven.
7. Implement the installer with interactive and non-interactive flows.
8. Add smoke tests for manifest discovery, launcher generation, install, uninstall, and command collision handling.
9. Document how to create a new recipe.
10. Publish the repo and update the installer URL.

## Verification

The first complete pass should prove:

- `recipes list` discovers local recipe manifests.
- `recipes install kx` creates a runnable `kx` command.
- Installed `kx` launches the Pi harness with the intended prompt and working directory.
- Existing commands are not overwritten silently.
- `recipes uninstall kx` removes generated launchers and state.
- The installer can run from a clean checkout.
- The README gives a new user enough information to install one recipe and author another.

## Non-Goals

- Do not build a general-purpose agent framework.
- Do not require users to adopt the author's full zsh config.
- Do not hard-code private local paths except as documented defaults that can be overridden.
- Do not assume every recipe must use the same runtime if Pi's model suggests otherwise.
- Do not migrate every existing zsh function before the first recipe contract is proven.

## Success Criteria

This goal is complete when `~/dev/recipes` contains a shareable repo scaffold with:

- A documented recipe directory convention.
- A working installer entrypoint.
- At least one functioning Pi harness migrated from the existing zsh wrapper pattern.
- Clear instructions for installing selected harnesses.
- Clear instructions for adding a new harness.
