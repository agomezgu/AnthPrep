When skills don't work, the problem usually falls into one of a few categories: the skill doesn't trigger, doesn't load, has conflicts, or fails at runtime.

#### **Key takeaways**

- Start with the **skills validator tool** — it catches structural problems before you spend time debugging other things
- If a skill **doesn't trigger**, the cause is almost always the description — add trigger phrases that match how you actually phrase requests
- If a skill **doesn't load**, check that `SKILL.md` is inside a named directory (not at the skills root) and the file name is exactly `SKILL.md`
- If the **wrong skill gets used**, your descriptions are too similar — make them more distinct
- For **runtime errors**, check dependencies, file permissions (`chmod +x`), and path separators (use forward slashes everywhere)



## **Skill Doesn't Load**

If your skill doesn't appear when you ask Claude "what skills are available," check these structural requirements:

- The `SKILL.md` file must be inside a named directory, not at the skills root
- The file name must be exactly `SKILL.md` — all caps on "SKILL", lowercase "md"

Run `claude --debug` to see loading errors. Look for messages mentioning your skill name. Sometimes this alone will point you straight to the problem.

## **Plugin Skills Not Appearing**

Installed a plugin but can't see its skills? Clear the cache, restart Claude Code, and reinstall.

If skills still don't appear after that, the plugin structure might be wrong. This is when the validator tool really earns its keep.

## **Runtime Errors**

The skill loads but fails during execution. A few common causes:

- **Missing dependencies:** If your skill uses external packages, they must be installed. Add dependency info to your skill description so Claude knows what's needed.
- **Permission issues:** Scripts need execute permission. Run `chmod +x` on any scripts your skill references.
- **Path separators:** Use forward slashes everywhere, even on Windows.



## **Quick Troubleshooting Checklist**

- **Not triggering?** Improve your description and add trigger phrases.
- **Not loading?** Check your path, file name, and YAML syntax.
- **Wrong skill used?** Make descriptions more distinct from each other.
- **Being shadowed?** Check the priority hierarchy and rename if needed.
- **Plugin skills missing?** Clear cache and reinstall.
- **Runtime failure?** Check dependencies, permissions, and paths.

