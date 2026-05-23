# canvas-mcp-skills

Claude Code/Desktop skills that work with the [canvas-mcp](https://github.com/jaymesdec/canvas-mcp-node) MCP server. Generic by design — adapt to whatever competency framework your school config defines (or no framework at all). Built at Franklin School (Jersey City, NJ) but reusable by any school using Canvas LMS.

## What's a skill?

A markdown file with frontmatter that lives at `~/.claude/skills/<skill-name>/SKILL.md`. Claude reads it when you trigger one of its trigger phrases ("plan a lesson", "create an assessment", etc.) and follows the documented workflow — calling the MCP's Canvas tools, generating content, publishing pages.

These skills don't run code; they're prompts that steer Claude's behavior. The MCP server (canvas-mcp) is what actually talks to Canvas.

## Prerequisites

- **Claude Desktop** (or any client that loads `~/.claude/skills/`)
- **canvas-mcp v0.3.11 or later** installed via [the .mcpb installer](https://github.com/jaymesdec/canvas-mcp-node/releases/latest)
- A school config loaded by the MCP (Franklin teachers: default; other schools: point `SCHOOL_CONFIG` at your own JSON — see canvas-mcp-node README)

## Install

```bash
# Clone the repo somewhere convenient
git clone https://github.com/jaymesdec/canvas-mcp-skills ~/code/canvas-mcp-skills

# Symlink the skills you want into Claude's skill directory
mkdir -p ~/.claude/skills
ln -s ~/code/canvas-mcp-skills/skills/plan-lesson ~/.claude/skills/plan-lesson
```

Or copy individual skill directories into `~/.claude/skills/` if you don't want a live symlink.

Restart Claude Desktop. The skill becomes active.

## Available skills

| Skill | What it does | Trigger phrases |
|---|---|---|
| [`plan-lesson`](skills/plan-lesson/) | Plan a Canvas lesson page using the school's lesson template. Generates content for all required slots; teacher reviews; published as a draft Canvas page. | "plan a lesson", "create a lesson", "draft a lesson page" |

More skills coming — `plan-assessment`, `plan-module`, `plan-course`, `publish-to-canvas` are on the roadmap.

## For other schools

These skills don't hardcode Franklin. They:

- Call `list_competencies` and adapt to whatever framework is configured (or skip competency-alignment if none)
- Call `list_page_templates` to discover what content slots the school's `lesson` / `assessment` / `default` templates expect
- Use whatever academic calendar / term names live in your school config

If your school config has different template names or competencies, the skills will pick them up automatically. No code changes needed.

## Contributing

Open issues / PRs at https://github.com/jaymesdec/canvas-mcp-skills.

When adding a new skill:

1. Create `skills/<name>/SKILL.md` with `name:` / `description:` frontmatter (the description is what triggers Claude).
2. Use `list_page_templates` and other tools to discover school-specific structure — don't hardcode slot names or competencies.
3. Add a row to the "Available skills" table in this README.
4. If the skill needs supporting files (templates, reference data), put them in `skills/<name>/` alongside the SKILL.md.

## License

MIT. See [LICENSE](LICENSE).
