# starter-pack

A personal **starter template + setup runbook** for Claude Code projects.

Goal: spin up a new project in seconds. Clone this repo (or use it as a GitHub template), open it in Claude Code, and the coding assistant is already configured — plugins enabled, permission prompts off, and behavioral guidelines active — with **no manual setup per project**.

---

## TL;DR — start a new project

**Option A — use as a GitHub template (recommended)**
```bash
# On GitHub: click "Use this template" → create new repo, OR:
gh repo create my-new-project --template <your-gh-username>/rey-learn-to-code --private --clone
cd my-new-project
# Open in Claude Code → done. Plugins auto-install on first launch.
```

**Option B — bolt the config onto an existing project**
```bash
cp -r rey-learn-to-code/.claude rey-learn-to-code/CLAUDE.md ./my-existing-project/
```

That is the entire per-project setup. Everything else is one-time, per machine.

---

## What's inside

```
rey-learn-to-code/
├── README.md              # this runbook
├── CLAUDE.md              # behavioral guidelines the assistant follows (Karpathy-adapted)
├── .gitignore             # sensible defaults + ignores machine-local Claude settings
└── .claude/
    └── settings.json      # committed, cloneable config: plugins + permissions
```

---

## What "clone and run" actually does

Be clear about what travels *in the repo* vs. what is *installed once per machine*.

| Thing | Where it lives | Travels with `git clone`? | Notes |
|-------|----------------|---------------------------|-------|
| Behavioral guidelines (`CLAUDE.md`) | repo root | ✅ yes | Auto-loaded as project instructions. |
| Project config (`.claude/settings.json`) | repo | ✅ yes | Declares plugins + permission mode. |
| Plugin **enablement** (which plugins to use) | `.claude/settings.json` | ✅ yes | Reconciled on project open. |
| Plugin **binaries** (superpowers, claude-mem) | `~/.claude/plugins/` | ⚙️ no | Auto-downloaded on first launch from the declared marketplaces (needs internet, one-time). |
| Permission bypass (`bypassPermissions`) | `.claude/settings.json` | ✅ yes | See one-time machine note below. |
| "Accept dangerous mode" acknowledgement | `~/.claude/settings.json` | ⚙️ no | One-time per machine (see below). |

**Bottom line:** on *your* machine (already set up), clone → open → it just works. On a *brand-new* machine, do the one-time setup below once, then every future clone just works.

---

## One-time machine setup (do once per computer)

1. **Install Claude Code** and sign in.

2. **Register the third-party marketplace** used by `claude-mem` (the official marketplace is known by default):
   ```bash
   claude plugin marketplace add thedotmack/claude-mem
   ```
   > You can skip this if you rely on the `extraKnownMarketplaces` entry already present in `.claude/settings.json` — Claude Code registers it automatically when the project opens.

3. **Let plugins install.** On first launch inside a project that has this `.claude/settings.json`, Claude Code installs and enables:
   - `superpowers@claude-plugins-official`
   - `claude-mem@thedotmack`

4. **Accept bypass mode once.** The first time `bypassPermissions` is used, Claude Code shows a one-time "I accept the risks" dialog. After accepting once, it stays silent on this machine for all projects.

That's it. From here on, new projects created from this template need zero setup.

---

## The configuration, explained

### `.claude/settings.json`
```json
{
  "permissions": { "defaultMode": "bypassPermissions" },
  "enabledPlugins": {
    "superpowers@claude-plugins-official": true,
    "claude-mem@thedotmack": true
  },
  "extraKnownMarketplaces": {
    "thedotmack": { "source": { "source": "github", "repo": "thedotmack/claude-mem" } }
  }
}
```

- **`permissions.defaultMode: bypassPermissions`** — the assistant runs every tool (bash, edits, etc.) without asking for approval.
  > ⚠️ **Security note.** This bypasses *all* permission checks, including destructive commands. It is committed here because this is a personal, single-owner template. If you ever share a project with others, they inherit bypass on clone — they can override it locally by setting a different `defaultMode` in their own `.claude/settings.local.json` (which is git-ignored).
- **`enabledPlugins`** — the plugins to auto-enable. Format: `plugin-name@marketplace-name`.
- **`extraKnownMarketplaces`** — registers the GitHub source for the `thedotmack` marketplace so the machine knows where to fetch `claude-mem` from.

### `CLAUDE.md`
Behavioral guidelines the assistant reads on every session. Summary:

- **§0 Execution style** — run tasks end-to-end; no confirmation questions on reversible actions.
- **§1 Think before coding** — state assumptions, surface tradeoffs; only ask once, up front, on genuinely ambiguous + costly decisions.
- **§2 Simplicity first** — minimum code that solves the problem; nothing speculative.
- **§3 Surgical changes** — touch only what the request requires; don't refactor unrelated code.
- **§4 Goal-driven execution** — turn tasks into verifiable success criteria and loop until they pass.

Adapted from [Andrej Karpathy's CLAUDE.md](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/CLAUDE.md).

---

## Plugins used

| Plugin | Marketplace | Purpose |
|--------|-------------|---------|
| `superpowers` | `claude-plugins-official` | Skill framework: brainstorming, TDD, systematic debugging, planning, and more. |
| `claude-mem` | `thedotmack` (`thedotmack/claude-mem`) | Persistent cross-session memory, code search, timeline/plan tooling. |

---

## Customizing per machine (optional)

To change behavior on *your* machine without editing the shared committed config, create a git-ignored override:

```jsonc
// .claude/settings.local.json  (ignored by .gitignore)
{
  "permissions": { "defaultMode": "acceptEdits" }   // e.g. safer than full bypass
}
```

Settings load order is: user (`~/.claude`) → project (`.claude/settings.json`) → local (`.claude/settings.local.json`), where later overrides earlier.
