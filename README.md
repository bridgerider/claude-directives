# Claude Directives

A Claude Code **plugin marketplace** distributing four rigorous, project-agnostic
**Operating Directives** as slash commands. Install once; update with one command.

| Command | What it does |
|---------|--------------|
| `/codethis` | **Engineering directive** — vertical slices, tests first, blast-radius-scaled hardening, gated deploys |
| `/fixthis` | **Debugging directive** — reproduce first, capture in a failing test, prove the root cause, fix minimally, prevent the class |
| `/auditthis` | **Code audit** (read-only) — machine checks first, findings scaled to blast radius, every finding backed by a concrete failure scenario, CONFIRMED vs PLAUSIBLE. Changes no code |
| `/researchthis` | **Research directive** — never assert without attribution, climb to primary sources, corroborate independently, calibrate confidence |

Each is **user-invoked only** (never auto-triggers) and takes your task as an argument,
e.g. `/fixthis the checkout total is off by a cent on multi-item carts`.

## Install

In Claude Code:

```
/plugin marketplace add bridgerider/claude-directives
/plugin install operating-directives@claude-directives
```

Then start a new session (or `/reload`) and type `/` — you'll see `/codethis`,
`/fixthis`, `/auditthis`, `/researchthis`.

Requires Claude Code v2.1.100 or later (plugin system). Works on macOS, Linux, and Windows.

## Updates

This plugin uses **commit-SHA versioning** — it declares no `version`, so **every commit
to this repo is a new version**. To pull the latest directives (and any newly added ones):

```
/plugin marketplace update
```

New directives added to this repo appear after that command — no reinstall needed.

## What's inside

```
claude-directives/
├── .claude-plugin/marketplace.json     ← the marketplace catalog
└── operating-directives/               ← the plugin
    ├── .claude-plugin/plugin.json      ← plugin manifest (no version → commit-SHA)
    └── commands/                       ← the four directives
        ├── codethis.md
        ├── fixthis.md
        ├── auditthis.md
        └── researchthis.md
```

## Uninstall

```
/plugin uninstall operating-directives@claude-directives
```

## License

See `LICENSE` in this repository.
