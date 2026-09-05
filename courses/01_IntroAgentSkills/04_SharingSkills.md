#### **Key takeaways**

- **Project skills** in `.claude/skills` are shared automatically through Git — anyone who clones the repo gets them
- **Plugins** let you distribute skills across repositories via marketplaces for broader community use
- **Enterprise managed settings** deploy skills organization-wide with the highest priority, ideal for mandatory standards and compliance
- **Subagents don't automatically see your skills** — you must explicitly list skills in a custom agent's frontmatter `skills` field
- Built-in agents (Explorer, Plan, Verify) **can't access skills at all** — only custom subagents defined in `.claude/agents` can

**NOTES**

The managed settings file supports features like `strictKnownMarketplaces` to control where plugins can be installed from:

```
"strictKnownMarketplaces": [
  {
    "source": "github",
    "repo": "acme-corp/approved-plugins"
  },
  {
    "source": "npm",
    "package": "@acme-corp/compliance-plugins"
  }
]
```

There are important distinctions to understand:

- **Built-in agents** (like Explorer, Plan, and Verify) can't access skills at all
- **Custom subagents** you define *can* use skills, but only when you explicitly list them
- Skills are loaded when the subagent starts, not on demand like in the main conversation

To create a custom subagent with skills, add an agent markdown file in `.claude/agents`. You can use the `/agents` command in Claude Code to create one interactively

The generated agent file includes a `skills` field that lists which skills to load. Here's what the frontmatter looks like:

```
---
name: frontend-security-accessibility-reviewer
description: "Use this agent when you need to review frontend code for accessibility..."
tools: Bash, Glob, Grep, Read, WebFetch, WebSearch, Skill...
model: sonnet
color: blue
skills: accessibility-audit, performance-check
---
```

