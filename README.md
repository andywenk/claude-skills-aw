# claude-skills

A collection of [Claude Skills](https://www.anthropic.com/news/claude-skills) for personal use, made shareable.

Each skill lives in its own directory with a `SKILL.md` and any supporting files. Skills are designed to be triggered inside Claude Code or other Claude-powered environments that read SKILL.md files.

## Skills

| Skill | Trigger | What it does |
|---|---|---|
| [`ai-news`](./ai-news/) | `/ai-news` | Curated weekly digest of AI news (research, industry, policy, tools) as an interactive widget with German summaries, focus weighting, and an auto-growing glossary with clickable definitions. |

More skills coming. The repo is a mono-repo — each subdirectory is one skill, fully self-contained.

## Two ways to use these skills

**A) Claude Code (terminal):** install via symlink (see below). Skills behave as proper Claude Code skills with separate config files (focus topics, glossaries, etc.).

**B) claude.ai Projects (browser, iPad, mobile apps):** where a skill has a `project-instructions.md`, copy its contents into a new Project's custom instructions. Skill then works on any device that can open the Project, with full interactive widget rendering.

Method B is useful when you don't have Claude Code available (e.g. on iPad) or when you want the full interactive UI that the terminal can't render.

## Installation

The recommended setup keeps the Git repo in a normal projects folder and symlinks individual skills into `~/.claude/skills/`. This way Claude finds the skill at the expected path, while you work inside a normal Git repo.

```bash
# 1. Clone the repo wherever you keep projects
git clone https://github.com/YOUR-USERNAME/claude-skills.git ~/Projects/claude-skills

# 2. Make sure the skills directory exists
mkdir -p ~/.claude/skills

# 3. Symlink the skill(s) you want
ln -s ~/Projects/claude-skills/ai-news ~/.claude/skills/ai-news
```

To install all skills at once:

```bash
for dir in ~/Projects/claude-skills/*/; do
  name=$(basename "$dir")
  [ -f "$dir/SKILL.md" ] && ln -sf "$dir" ~/.claude/skills/"$name"
done
```

Verify the symlink:

```bash
ls -l ~/.claude/skills/
```

## First run

Some skills use user-specific configuration files (e.g. `FOCUS.md` for `ai-news`). These files are not in the repo — only their `.example` templates are.

On first run, the skill copies the `.example` file to the live filename if missing, so you can start immediately and then customize. The live files are gitignored, so your private content stays out of the repo.

## Updating

Since each skill is a symlink to the repo:

```bash
cd ~/Projects/claude-skills
git pull
```

Changes apply immediately — no re-install needed.

## Contributing your own skills

This repo is structured to grow. To add a skill:

1. Create a new top-level directory: `your-skill-name/`
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`) and the skill instructions.
3. Optionally add `references/`, `scripts/`, or other supporting files.
4. If the skill uses user-specific config, ship `.example` templates and add the live filenames to `.gitignore`.
5. Add an entry to the Skills table above.

See Anthropic's [skill-creator skill](https://github.com/anthropics/skills) for guidance on structuring SKILL.md files.

## License

Apache License 2.0 — see [LICENSE](./LICENSE).
