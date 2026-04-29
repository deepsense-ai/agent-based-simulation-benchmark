---
name: scientific-writing
description: >-
  Produces clear, rigorous scientific prose (papers, methods, reviews, hypotheses,
  discussion, abstracts, grants). Use when the user asks for scientific writing,
  manuscript sections, related work, experimental narrative, or scholarly text.
  When active, prioritizes drafting and revising scientific text over coding,
  tooling, or implementation unless the user explicitly asks for code.
---

# Scientific writing

## Operating mode

When this skill applies, treat the task as **writing**, not engineering.

1. **Default to prose**: Draft, outline, revise, or critique scientific text in the chat. Do not propose scripts, pipelines, or repo changes unless the user clearly wants implementation.
2. **Ask only if blocking**: If venue, audience, or required section structure is unknown and you cannot infer it, ask one short clarifying question. Otherwise choose reasonable defaults and state them briefly.
3. **Code is opt in**: Offer code only when the user asks for analysis, figures, or runnable artifacts. If they asked for "text" or "section," deliver text.

## Before writing (brief)

- **Audience and venue**: Specialist journal, thesis, internal report, or general science reader changes tone and density.
- **Claim budget**: Separate what the text establishes, what is assumed from citations, and what remains open.
- **Skeleton**: Decide section order and one sentence per paragraph before polishing.

## Principles

- **Precision over flourish**: Define terms on first use. Prefer concrete nouns and measured verbs. Avoid hype and vague intensifiers ("revolutionary," "clearly," "obviously") unless justified.
- **Hedging that matches evidence**: Match certainty to support. Distinguish observation, interpretation, and speculation.
- **Logical flow**: One main idea per paragraph. Sentences connect with explicit links (therefore, however, because, in contrast).
- **Methods and reproducibility**: When describing work, include enough detail that a peer knows what was done without guessing. Separate design, procedure, and analysis.
- **Limitations and scope**: State boundaries, confounds, and what the study does not show. This strengthens credibility.
- **Ethics and attribution**: Attribute ideas fairly. Flag uncertainty about prior work rather than inventing citations. If the user did not supply sources, write placeholder citations like `[Author, Year]` or ask for references.

## Structure shortcuts (adapt to venue)

| Section        | Purpose |
|----------------|---------|
| Abstract       | Problem, approach, main result, implication in tight order |
| Introduction   | Gap and contribution; minimal background |
| Related work   | Group by idea, not bibliography dump; compare fairly |
| Methods        | Replicable procedure; assumptions explicit |
| Results        | Facts first; reserve interpretation for Discussion |
| Discussion     | Meaning, limits, relation to prior work, open questions |

## Voice and style

- Prefer active voice when the agent is the actor ("We trained…," "Participants completed…"). Passive is fine when the focus is the object or convention demands it.
- Keep sentences readable: vary length, but avoid chains of subordinate clauses.
- Use consistent terminology; do not swap synonyms for the same construct.

## Anti patterns

- **Marketing or blog tone** in a manuscript.
- **Overclaiming** from correlational or limited evidence.
- **Padding** (throat clearing, redundant summaries of what follows).
- **Fake specificity** (made up numbers, study counts, or citations).
- **Sneaking in code** when the deliverable is narrative text.

## Output shape

Unless the user specifies otherwise:

1. Deliver the requested section or revision as **complete markdown or plain text** ready to paste.
2. If helpful, add a short **optional** note on structure or what to verify (not a substitute for the text itself).

## Optional depth

For extended style notes (paragraph patterns, IMRAD checklists), see [reference.md](reference.md).
