# Lazy-Loading Documentation with Skills

## Problem (Issue #37)

The current modular documentation structure loads all referenced documentation files into context immediately at conversation start, consuming 2,131 lines of tokens per conversation regardless of task relevance.

### Current Token Cost
- CLAUDE.md: 163 lines
- Six supporting docs: 1,968 lines
- **Total:** 2,131 lines loaded upfront every conversation

### Why This Happens
1. **@ file references load all content** - The "@" file reference system in Claude Code is not lazy-loaded; all referenced files appear as separate context entries
2. **Subdirectory lazy-loading is broken** - The intended feature allowing conditional loading based on directory context doesn't function
3. **No organizational savings** - The modular v2.0.0 structure offers organizational benefits only, providing zero token savings versus the previous monolithic approach

## Solution: Skills Instead of @ References

Skills provide true lazy-loading with 0 lines upfront context cost.

### How It Works
1. **@ file references:** Load entire file content into context immediately when conversation starts
2. **Skills:** Load 0 lines upfront, then 20-50 lines on-demand when user queries the skill

### Token Comparison
| Approach | Upfront Cost | Per Query | Total (5 queries) |
|----------|--------------|-----------|-------------------|
| @ references | 2,131 lines | 0 lines | 2,131 lines |
| Skills | 0 lines | 20-50 lines | 100-250 lines |
| **Savings** | **100%** | n/a | **88-95%** |

## Example Implementation: tdd-reference.md

See `claude/.claude/skills/tdd-reference.md` for a working example that demonstrates:

- **Indexed guideline system:** Load only relevant sections via user queries
- **Quick reference cards:** 20-50 line summaries for RED/GREEN/REFACTOR phases
- **Full docs still accessible:** Complete documentation remains available on-demand
- **95%+ token reduction:** From 2,131 lines to 20-50 lines per query

### Usage Pattern
```markdown
---
description: On-demand guideline access without loading full documentation
---

# Skill Content
Provides indexed access to guidelines:
- User asks: "What are the RED phase guidelines?"
- Skill loads: Only the 30-line RED section (not all 2,131 lines)
- Zero upfront cost, only load what's needed when needed
```

### How Users Interact With Skills

**In Claude Code (Desktop/CLI):**
```
User: "What are the RED phase guidelines?"
Claude: [Loads only RED section from tdd-reference skill - ~30 lines]
        [Provides answer based on that specific section]

User: "What about REFACTOR phase best practices?"
Claude: [Loads only REFACTOR section - ~25 lines]
        [Previous RED section no longer in context if not needed]
```

**Token Math:**
- Without skills: 2,131 lines loaded at conversation start
- With skills: 0 lines + 30 lines + 25 lines = 55 lines total
- **Savings: 97.4%**

## Implementation Pattern for Other Documentation

Apply this pattern to your existing docs:

### Step 1: Convert Large Docs to Skills

For each major documentation file (testing.md, typescript.md, workflow.md):

1. Create corresponding skill file in `claude/.claude/skills/`
2. Add minimal YAML frontmatter:
```yaml
---
description: [Brief description of what this skill provides]
---
```

3. Organize content with clear section headers:
```markdown
# Testing Guidelines Skill

## Quick Reference
[20-30 line summary for quick access]

## Unit Testing
[Detailed unit testing guidelines]

## Integration Testing
[Detailed integration testing guidelines]

## Test Patterns
[Common patterns and examples]
```

### Step 2: Update CLAUDE.md

**Before (loads everything):**
```markdown
For detailed testing guidelines, see @testing.md
For TypeScript patterns, see @typescript.md
For workflow details, see @workflow.md
```
**Result:** All 1,968 lines loaded immediately

**After (loads nothing):**
```markdown
For testing guidance, ask about the testing skill
For TypeScript patterns, ask about the typescript skill
For workflow details, ask about the workflow skill
```
**Result:** 0 lines loaded upfront

### Step 3: Create Index Sections

Organize each skill by topic for selective loading:

```markdown
# TypeScript Skill

## INDEX
1. Strict Mode Configuration
2. Type Safety Patterns
3. Generics Best Practices
4. Error Handling
5. Async/Await Patterns

## [Each section detailed below]
```

Users can ask: "Show me section 2 from typescript skill" or "What are the error handling patterns?"

## Migration Path

### Phase 1: High-Volume Docs (Week 1)
Convert the largest documentation files to skills:
- testing.md → testing-skill.md
- typescript.md → typescript-skill.md
- workflow.md → workflow-skill.md

**Expected savings:** 60-70% token reduction

### Phase 2: Update CLAUDE.md (Week 2)
- Remove @ references to converted files
- Add skill interaction instructions
- Keep only essential principles (~150-200 lines)

**Expected savings:** 85-90% token reduction

### Phase 3: Measurement (Week 3)
- Use context inspection to verify token counts
- Gather user feedback on skill interaction
- Adjust skill organization based on common queries

### Phase 4: Documentation & Best Practices (Week 4)
- Document skill-based architecture
- Create contribution guidelines for new skills
- Share learnings with community

## Technical Details

### Skill File Structure
```
claude/.claude/skills/
├── tdd-reference.md          (example from this PR)
├── testing-skill.md          (future: testing guidelines)
├── typescript-skill.md       (future: TS patterns)
├── workflow-skill.md         (future: workflow docs)
└── README.md                 (skill index)
```

### Frontmatter Requirements
Minimal YAML frontmatter - just description:
```yaml
---
description: Brief description for skill selection UI
---
```

**Note:** Claude Code supports additional fields (name, tools, model, category, tags), but Claude.ai web app only accepts `description` field. Use minimal frontmatter for maximum compatibility.

### Content Organization Best Practices

1. **Start with quick reference** (20-30 lines)
2. **Index all major sections** for easy navigation
3. **Use clear headers** for selective loading
4. **Include examples** inline rather than separate files
5. **Keep related content together** (easier to load as unit)

## Expected Results

### Token Reduction
- **Upfront:** 100% reduction (2,131 → 0 lines)
- **Per conversation:** 85-95% reduction (typical 5-query conversation: 2,131 → 100-250 lines)
- **Scalability:** Add unlimited docs without increasing upfront cost

### Performance Improvements
- **Faster conversation starts** (less initial context loading)
- **More relevant context** (only load what's needed)
- **Better Claude responses** (less noise in context)
- **Longer conversations** (more room for actual code/discussion)

### User Experience
- **On-demand access** to full documentation depth
- **No manual file navigation** (Claude handles skill selection)
- **Consistent interface** across all documentation
- **Discovery through conversation** ("What skills are available?")

## Platform Compatibility

### Claude Code (Desktop & CLI)
✅ Full skill support
✅ Skills appear in UI
✅ On-demand loading
✅ Extended YAML frontmatter

### Claude.ai Web App
⚠️  No custom skills support
✅ Alternative: Use as Project Knowledge
✅ Upload skill files as reference docs
❌ Won't get lazy-loading benefits

**Recommendation:** Design skills for Claude Code, document Project Knowledge fallback for web users.

## Measuring Success

### Before Migration
```bash
# Count lines in context at conversation start
wc -l claude/.claude/CLAUDE.md claude/.claude/docs/*.md
# Result: ~2,131 lines
```

### After Migration
```bash
# Start conversation, check context
# Result: ~150-200 lines (CLAUDE.md only)

# After 5 skill queries
# Result: ~350-450 lines total (still 80%+ savings)
```

### Metrics to Track
1. **Upfront token count** (context at conversation start)
2. **Average tokens per conversation** (after typical usage)
3. **User satisfaction** (can users find what they need?)
4. **Query patterns** (which skills/sections most accessed?)

## Frequently Asked Questions

### Q: Won't users miss having all docs in context?
A: Skills provide the same content on-demand. Users ask for what they need, Claude loads just that section. More efficient than loading everything hoping the right info is there.

### Q: How do users know what skills are available?
A: CLAUDE.md can list available skills, or users can ask "What skills do you have?" Claude can enumerate skills from the directory.

### Q: What if users need multiple sections?
A: Claude can load multiple sections if needed. Even loading 5-6 sections (250 lines) is still 88% savings vs loading everything (2,131 lines).

### Q: Can skills reference each other?
A: Yes, but be careful. Cross-referencing skills can negate the token savings. Keep skills self-contained where possible.

### Q: What about code examples?
A: Include examples inline in skills. Small examples (5-20 lines) are fine. For large examples, consider separate skill or load on-demand.

## Contributing New Skills

When creating new skills:

1. **Start with user needs** - What questions do they ask most?
2. **Organize by topic** - Group related info together
3. **Index major sections** - Enable selective loading
4. **Test token counts** - Verify savings vs @ references
5. **Document in CLAUDE.md** - Make skill discoverable

## Credits

This lazy-loading approach was developed by @jackspace based on citypaul's excellent TDD documentation structure, specifically to address issue #37's token optimization goals.

The tdd-reference.md skill demonstrates the pattern with a real-world example showing 95%+ token reduction in practice.
