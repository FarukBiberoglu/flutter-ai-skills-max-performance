# Flutter AI Skills

A collection of Claude Code skills built to speed up Flutter development.

## Skills

- **flutter-ui-reviewer** — Reviews Flutter UI code and suggests improvements.
- **flutter-responsive-layout-reviewer** — Reviews layout code for responsiveness, especially `Flexible` / `Expanded` usage; catches overflow and fixed-size bugs.
- **flutter-performance-reviewer** — Detects performance bottlenecks (rebuilds, jank, memory leaks) and suggests optimizations.
- **clean-architecture-reviewer** — Checks conformance to the project's Clean Architecture layers.
- **firebase-flutter-helper** — Reviews Firebase integration (Auth, Cloud Messaging) against the architecture.
- **flutter-theme-reviewer** — Reviews and scaffolds the theme/token system (layered color, typography, spacing).

## Structure

```
flutter-ai-skills/
├── README.md
├── CLAUDE.md                   # LLM behavioral guidelines (think / simplicity / surgical / goal-driven)
├── skills/
│   ├── flutter-ui-reviewer/
│   ├── flutter-responsive-layout-reviewer/
│   ├── flutter-performance-reviewer/
│   ├── clean-architecture-reviewer/
│   ├── firebase-flutter-helper/
│   └── flutter-theme-reviewer/
└── examples/
    └── analysis_options.yaml   # Recommended lint rules
```

## CLAUDE.md

While running, every skill is grounded in the 4 core rules in [CLAUDE.md](CLAUDE.md):

1. **Think Before Coding** — state your assumptions; if uncertain, ask.
2. **Simplicity First** — minimum code, no unrequested features/abstractions.
3. **Surgical Changes** — touch only what was asked; don't "improve" adjacent code.
4. **Goal-Driven Execution** — define success criteria and loop until verified.

## Lint rules

The `analysis_options.yaml` file the skills are based on lives under `examples/`. Copy it into your own Flutter project:

```bash
cp flutter-ai-skills/examples/analysis_options.yaml <your-project>/analysis_options.yaml
```

## Requirements

These are [Claude Code](https://claude.com/claude-code) skills. You need Claude Code (CLI, desktop, or IDE extension) installed. Each skill is a folder containing a `SKILL.md` with `name` + `description` frontmatter — Claude Code auto-discovers it once the folder is placed in a skills directory.

## Installation

Clone the repo, then install the skills into one of the directories Claude Code scans.

**Option A — Personal (available in all your projects):**

```bash
git clone https://github.com/FarukBiberoglu/flutter-ai-skills-max-performance.git
mkdir -p ~/.claude/skills
cp -r flutter-ai-skills-max-performance/skills/* ~/.claude/skills/
```

**Option B — Per project (share with your team via git):**

```bash
# from your Flutter project's root
mkdir -p .claude/skills
cp -r /path/to/flutter-ai-skills-max-performance/skills/* .claude/skills/
git add .claude/skills && git commit -m "Add Flutter review skills"
```

Everyone who clones the project then gets the same skills.

> Also copy `examples/analysis_options.yaml` into your Flutter project root — several skills (`flutter-ui-reviewer`, `flutter-responsive-layout-reviewer`) require it and will refuse to run without it.

## How to use

Once installed, invoke a skill in Claude Code two ways:

- **Automatically** — just describe the task; Claude picks the matching skill from its `description`. e.g. *"review this screen for responsive layout issues"* → `flutter-responsive-layout-reviewer`.
- **Explicitly** — type the slash command, which is the skill's `name`:
  - `/flutter-ui-reviewer`
  - `/flutter-responsive-layout-reviewer`
  - `/flutter-performance-reviewer`
  - `/clean-architecture-reviewer`
  - `/firebase-flutter-helper`
  - `/flutter-theme-reviewer`

Each skill reads your changed files (`git diff`), checks them against its rules, and reports findings as `path:line — [TAG] — problem — suggested fix`. They review and suggest; they don't edit your code on their own.

**No install, one-off use:** you can also point Claude at a file directly — *"review this widget against `skills/flutter-ui-reviewer/SKILL.md`"* — without copying anything.
