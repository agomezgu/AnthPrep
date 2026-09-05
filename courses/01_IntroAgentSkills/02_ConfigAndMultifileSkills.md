# Key takeaways

-name and description are required — allowed-tools and model are optional but powerful additions

- A good description answers two questions: What does the skill do? When should Claude use it?
- allowed-tools restricts which tools Claude can use when the skill is active — useful for read-only or security-sensitive workflows
- If you omit `allowed-tools` entirely, the skill doesn't restrict anything. Claude uses its normal permission model.
- Progressive disclosure: keep SKILL.md under 500 lines and link to supporting files (references, scripts, assets) that Claude reads only when needed
- Scripts execute without loading their contents into context — only the output consumes tokens, keeping context efficient



# Skill Metadata Fields

The agent skills open standard supports several fields in the SKILL.md frontmatter. Two are required, and the rest are optional:

- **name** (required) — Identifies your skill. Use lowercase letters, numbers, and hyphens only. Maximum 64 characters. Should match your directory name.
- **description** (required) — Tells Claude when to use the skill. Maximum 1,024 characters. This is the most important field because Claude uses it for matching.
- **allowed-tools** (optional) — Restricts which tools Claude can use when the skill is active.
- **model** (optional) — Specifies which Claude model to use for the skill.

## **Progressive Disclosure**

The open standard suggests organizing your skill directory with:

- **scripts/** — Executable code
- **references/** — Additional documentation
- **assets/** — Images, templates, or other data files

## **Using Scripts Efficiently**

Scripts in your skill directory can run without loading their contents into context. The script executes and only the output consumes tokens. The key instruction to include in your [SKILL.md](http://SKILL.md) is to tell Claude to *run* the script, not *read* it.

This is particularly useful for:

- Environment validation
- Data transformations that need to be consistent
- Operations that are more reliable as tested code than generated code

