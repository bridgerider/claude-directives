# Claude Directives

A Claude Code **plugin marketplace** distributing five rigorous, project-agnostic
**Operating Directives** as slash commands. Install once; update with one command.

Installed via this plugin, the commands are **namespaced under `go:`**:

| Command | What it does |
|---------|--------------|
| `/go:planthis` | **Planning directive** — build an evidence-grounded, reviewable plan before any execution; two modes (software-implementation, precursor to `/codethis`; and general initiative). Writes no code |
| `/go:codethis` | **Engineering directive** — vertical slices, tests first, blast-radius-scaled hardening, gated deploys |
| `/go:fixthis` | **Debugging directive** — reproduce first, capture in a failing test, prove the root cause, fix minimally, prevent the class |
| `/go:auditthis` | **Code audit** (read-only) — machine checks first, findings scaled to blast radius, every finding backed by a concrete failure scenario, CONFIRMED vs PLAUSIBLE. Changes no code |
| `/go:researchthis` | **Research directive** — never assert without attribution, climb to primary sources, corroborate independently, calibrate confidence |

Each is **user-invoked only** (never auto-triggers) and takes your task as an argument,
e.g. `/go:fixthis the checkout total is off by a cent on multi-item carts`.

## Install

In Claude Code (or Cowork):

```
/plugin marketplace add https://github.com/bridgerider/claude-directives.git
/plugin install go@claude-directives
```

Then run `/reload-plugins` (or start a new session) and type `/go:` — you'll see the five commands.

> **Use the full HTTPS URL above — not the `bridgerider/claude-directives` shorthand.**
> Claude Code clones GitHub `owner/repo` shorthand sources **over SSH by default**. On a machine
> that has never connected to GitHub over SSH that fails immediately with
> `No ED25519 host key is known for github.com` / `Host key verification failed`, and even after
> you fix `known_hosts` an SSH clone still needs a GitHub SSH key — despite this repo being public.
> The HTTPS URL needs neither. (The `.git` suffix matters: without it Claude Code treats the URL as
> a link to a hosted `marketplace.json` rather than a repo to clone.) If you would rather keep using
> the shorthand, set `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` in your environment first.

Requires Claude Code v2.1.100 or later (plugin system). Works on macOS, Linux, and Windows.

## Updates

This plugin uses **commit-SHA versioning** — it declares no `version`, so **every commit
to this repo is a new version**. To pull the latest directives (and any newly added ones):

```
/plugin marketplace update
```

Or toggle **Sync automatically** on the marketplace (`/plugin` → Marketplaces) for
hands-off updates on session start. New directives added to this repo appear after an
update — no reinstall needed.

## What's inside

```
claude-directives/
├── .claude-plugin/marketplace.json     ← the marketplace catalog (marketplace name: claude-directives)
└── go/                                 ← the plugin (name: go → commands are /go:*)
    ├── .claude-plugin/plugin.json      ← plugin manifest (no version → commit-SHA)
    └── commands/                       ← the five directives
        ├── planthis.md
        ├── codethis.md
        ├── fixthis.md
        ├── auditthis.md
        └── researchthis.md
```

## Uninstall

```
/plugin uninstall go@claude-directives
```

## License

MIT — see `LICENSE`.
