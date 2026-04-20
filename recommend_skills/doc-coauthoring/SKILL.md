---
name: doc-coauthoring
description: Guide users through a structured workflow for co-authoring documentation. Use when user wants to write documentation, proposals, technical specs, decision docs, or similar structured content. This workflow helps users efficiently transfer context, refine content through iteration, and verify the doc works for readers. Trigger when user mentions writing docs, creating proposals, drafting specs, or similar documentation tasks.
---

# Doc Co-Authoring Workflow

This skill provides a structured workflow for guiding users through collaborative document creation. Act as an active guide, walking users through three stages: Context Gathering, Refinement & Structure, and Reader Testing.

## Stage 1: Context Gathering

**Goal:** Close the gap between what the user knows and what Claude knows.

### Initial Questions
1. What type of document is this? (e.g., technical spec, decision doc, proposal)
2. Who's the primary audience?
3. What's the desired impact when someone reads this?
4. Is there a template or specific format to follow?
5. Any other constraints or context to know?

### Info Dumping
Encourage the user to dump all context they have:
- Background on the project/problem
- Related team discussions or documents
- Why alternative solutions aren't being used
- Organizational context
- Timeline pressures or constraints
- Technical architecture or dependencies
- Stakeholder concerns

Don't worry about organizing — just get it all out.

### Clarifying Questions
After initial dump, generate 5-10 numbered questions based on gaps. User can answer in shorthand.

**Exit condition:** Sufficient context when edge cases and trade-offs can be discussed without needing basics explained.

## Stage 2: Refinement & Structure

**Goal:** Build the document section by section through brainstorming, curation, and iterative refinement.

For each section:
1. **Clarifying Questions** — Ask 5-10 questions about what to include
2. **Brainstorming** — Generate 5-20 numbered options
3. **Curation** — User indicates keep/remove/combine
4. **Gap Check** — Ask if anything important is missing
5. **Drafting** — Write the section based on selections
6. **Iterative Refinement** — Make surgical edits based on feedback

Start with whichever section has the most unknowns.

### Quality Checking
After 3 consecutive iterations with no substantial changes, ask if anything can be removed without losing important information.

### Near Completion
At 80%+ sections done, re-read entire document and check for:
- Flow and consistency across sections
- Redundancy or contradictions
- Generic filler
- Whether every sentence carries weight

## Stage 3: Reader Testing

**Goal:** Test the document with a fresh perspective to verify it works for readers.

### Steps:
1. **Predict Reader Questions** — Generate 5-10 realistic questions readers would ask
2. **Test** — If sub-agents available, test with fresh Claude instance. Otherwise, guide user to test in a new conversation.
3. **Additional Checks** — Check for ambiguity, false assumptions, contradictions
4. **Report and Fix** — Loop back to refinement for problematic sections

### Exit Condition
When Reader Claude consistently answers questions correctly and doesn't surface new gaps.

## Final Review
1. Recommend a final read-through by the user
2. Suggest double-checking facts, links, technical details
3. Verify it achieves the intended impact
