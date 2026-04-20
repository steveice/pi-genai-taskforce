---
name: skill-creator
description: Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to create a skill from scratch, edit or optimize an existing skill, run evals to test a skill, benchmark skill performance, or optimize a skill's description for better triggering accuracy.
---

# Skill Creator

A skill for creating new skills and iteratively improving them.

## Core Loop

1. Decide what you want the skill to do and roughly how it should do it
2. Write a draft of the skill
3. Create test prompts and run Claude-with-access-to-the-skill on them
4. Evaluate the results qualitatively and quantitatively
5. Rewrite the skill based on feedback
6. Repeat until satisfied
7. Expand the test set and try again at larger scale

## Creating a Skill

### Capture Intent

1. What should this skill enable Claude to do?
2. When should this skill trigger? (what user phrases/contexts)
3. What's the expected output format?
4. Should we set up test cases to verify the skill works?

### Skill Structure

```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (name, description required)
│   └── Markdown instructions
└── Bundled Resources (optional)
    ├── scripts/    - Executable code for deterministic/repetitive tasks
    ├── references/ - Docs loaded into context as needed
    └── assets/     - Files used in output (templates, icons, fonts)
```

### SKILL.md Frontmatter

```yaml
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---
```

The `description` is the primary triggering mechanism — include both what the skill does AND specific contexts for when to use it. Make descriptions slightly "pushy" to combat undertriggering.

### Writing Patterns

- Use imperative form in instructions
- Explain **why** things are important rather than heavy-handed MUSTs
- Use theory of mind — make the skill general, not narrow to specific examples
- Keep SKILL.md under 500 lines; use reference files for overflow
- For large reference files (>300 lines), include a table of contents

### Progressive Disclosure

1. **Metadata** (name + description) — Always in context (~100 words)
2. **SKILL.md body** — In context whenever skill triggers (<500 lines ideal)
3. **Bundled resources** — As needed (unlimited, scripts can execute without loading)

## Testing

Create 2-3 realistic test prompts — the kind of thing a real user would actually say. Save to `evals/evals.json`:

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "User's task prompt",
      "expected_output": "Description of expected result",
      "files": []
    }
  ]
}
```

## Description Optimization

The description field determines whether Claude invokes a skill. To optimize:

1. Generate 20 eval queries — mix of should-trigger and should-not-trigger
2. Make queries realistic with detail (file paths, personal context, casual speech)
3. For should-not-trigger, focus on near-misses — queries that share keywords but need something different
4. Test and iterate on the description based on trigger accuracy

## Tips

- Start by writing a draft, then look at it with fresh eyes and improve
- Generalize from feedback — don't overfit to specific test examples
- Keep the prompt lean — remove things that aren't pulling their weight
- Look for repeated work across test cases and bundle common scripts
- Explain the why behind instructions — LLMs respond better to reasoning than rigid rules
