> **Note:** This repository is my personal skills collection, organized in the style of `anthropics/skills`. For information about the Agent Skills standard, see [agentskills.io](https://agentskills.io).

# Skills

Skills are folders of instructions, scripts, and resources that an agent can load dynamically to improve performance on specialized tasks. This repository collects my reusable skills, including custom workflows and selected reference skills copied from `anthropics/skills` for alignment and study.

For more information, check out:
- [Agent Skills specification](https://agentskills.io/specification)
- [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

# About This Repository

This repository contains a small but growing set of skills that I want to reuse across projects.

Each skill is self-contained in its own folder with a `SKILL.md` file containing the instructions and metadata that the agent uses. Some skills may also include examples, scripts, templates, or other resources.

Currently included:

- `skills/awesome-html-note-agent` — my custom workflow for turning a topic into an interactive single-file HTML note, then iterating through Playwright review and refinement.
- `skills/frontend-design` — copied from `anthropics/skills` as a reference skill.
- `skills/internal-comms` — copied from `anthropics/skills` as a reference skill, including examples.

## Disclaimer

**These skills are provided for personal use, experimentation, and educational purposes.** Always test skills thoroughly in your own environment before relying on them for important tasks.

Copied third-party skill content remains subject to its original license terms. See `THIRD_PARTY_NOTICES.md` and any `LICENSE.txt` files inside copied skill folders.

# Skill Sets

- [./skills](./skills): Reusable skill examples and workflows
- [./spec](./spec): The Agent Skills specification pointer
- [./template](./template): Basic skill template

# Install and Use

## Local installation

Clone this repository anywhere on your machine:

```bash
git clone https://github.com/TianCai19/skills.git
```

Then copy or symlink the skill you want into your local skills directory. For example:

```bash
mkdir -p ~/.trae/skills
cp -R ./skills/awesome-html-note-agent ~/.trae/skills/
```

Or use a symlink during active development:

```bash
mkdir -p ~/.trae/skills
ln -s "$(pwd)/skills/awesome-html-note-agent" ~/.trae/skills/awesome-html-note-agent
```

After installation, the agent can use the skill when the `name` and `description` match the user request.

## Using a copied reference skill

For example, to install the copied `frontend-design` skill locally:

```bash
mkdir -p ~/.trae/skills
cp -R ./skills/frontend-design ~/.trae/skills/
```

## Claude.ai / API

If your client supports importing custom skills, upload the desired skill folder or use the contents of `SKILL.md` as the basis for your hosted skill definition.

# Creating a Basic Skill

Skills are simple to create - just a folder with a `SKILL.md` file containing YAML frontmatter and instructions. You can use the **template-skill** in this repository as a starting point:

```markdown
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---

# My Skill Name

[Add your instructions here that Claude will follow when this skill is active]

## Examples
- Example usage 1
- Example usage 2

## Guidelines
- Guideline 1
- Guideline 2
```

The frontmatter requires only two fields:
- `name` - A unique identifier for your skill (lowercase, hyphens for spaces)
- `description` - A complete description of what the skill does and when to use it

# Attribution

This repository includes selected reference skills copied from `anthropics/skills` to help keep the repository structure aligned with upstream conventions.
