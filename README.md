# OpenCode Skills

Custom skills for [OpenCode](https://opencode.ai) - an open source AI coding agent.

## Installation

Clone this repo and symlink to your OpenCode skills directory:

```bash
git clone git@github.com:spartandingo/opencode-skills.git ~/projects/opencode-skills

# Symlink to OpenCode config
ln -s ~/projects/opencode-skills/skills/* ~/.config/opencode/skills/
```

Or symlink the entire skills directory:

```bash
ln -s ~/projects/opencode-skills/skills ~/.config/opencode/skills
```

## Available Skills

| Skill | Description |
|-------|-------------|
| `staff-pr-review` | Review PRs with the rigor of a senior staff engineer |

## Available Commands

Copy commands to your OpenCode commands directory:

```bash
cp ~/projects/opencode-skills/commands/* ~/.config/opencode/commands/
```

| Command | Usage | Description |
|---------|-------|-------------|
| `/review-pr` | `/review-pr 123` | Review PR #123 like a hard-ass staff engineer |

## Usage

### Skill (loaded on-demand)

The agent can load the skill when needed:

```
Review this PR using the staff-pr-review skill
```

### Slash Command (direct invocation)

```
/review-pr 293
```

## License

MIT
