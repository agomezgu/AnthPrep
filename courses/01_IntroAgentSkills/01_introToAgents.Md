# Key takeaways
- A skill is a directory containing a SKILL.md file with metadata (name, description) in frontmatter and instructions below
- Claude loads only skill names and descriptions at startup, then matches incoming requests against those descriptions using semantic matching
- You get a confirmation prompt before Claude loads the full skill content into context
- Priority for name conflicts: Enterprise → Personal → Project → Plugins
- To update a skill, edit its SKILL.md. To remove one, delete its directory. Always restart Claude Code for changes to take effect

# Notes

- The name identifies your skill. The description tells Claude when to use it — this is the matching criteria. Everything after the second set of dashes is the instructions Claude follows when the skill is activated.

# Skill Priority
If you clone a repository that has a skill with the same name as one of your personal skills, which one wins? There's a clear priority order:

1.  Enterprise — managed settings, highest priority
2. Personal — your home directory (~/.claude/skills)
3. Project — the .claude/skills directory inside a repository
4. Plugins — installed plugins, lowest priority