---
name: social-response-writer
description: Drafts high-signal LinkedIn and Twitter/X replies for an AI agent observability founder. Converts social posts about AI agent failures, debugging pain, or production monitoring into comments designed to start conversations that lead to design partner pilots. Use when asked to "write a reply", "draft a comment", "respond to this post", or "write a social response" about AI agents, observability, or LLM tooling.
---

# Social Response Writer

Draft social replies that convert into discovery calls. The goal is never a like, it is a thread that turns into a DM, then a 15-minute call, then a design partner pilot.

**Positioning:** "AI agent observability that tells you *why* it failed, not just that it did." Entry price: $49/month.

**Feature hooks and competitive context:** See [references/differentiation_features.md](references/differentiation_features.md)
**Canonical examples:** See [references/example_responses.md](references/example_responses.md)

---

## When to Respond

- Post describes agent failures, debugging pain, monitoring challenges, or production surprises
- Thought leader or framework maintainer posts about observability (LangChain, LangGraph, CrewAI, AutoGen)
- Post leaves a gap or does not fully address a problem

---

## Response Workflow

### Step 1: Read the Full Post (and any linked article)

Check whether the post already covers the differentiator being considered. If it does, pick a different hook. Adding something the article already says destroys credibility.

### Step 2: Pick One Hook

From [references/differentiation_features.md](references/differentiation_features.md), choose the single most relevant differentiator. One. Never list multiple.

If the post mentions a competitor claiming one of these features, fetch and verify before commenting.

### Step 3: Choose a Response Style

Pick one style. Do not mix them.

#### Style A: Observation-led (default)

1. A short observation that extends or deepens the post's point (1-2 sentences). Do not restate what was said.
2. The pain that hook addresses, grounded in real developer experience (1-2 sentences).
3. One sentence stating what is being built toward, direct, not vague.

#### Style B: Curiosity-led

Use when the post describes a workflow or tool they built.

1. Open with a genuine question about how they currently handle the failure or debugging case the hook addresses. Make it specific to their setup, not generic.
2. Name the underlying pain: why that failure mode is hard to catch as scale increases or humans are removed from the loop (1-2 sentences).
3. One sentence tying it to what is being built, direct, not vague.

#### Style C: Acknowledge and ask

Use when the post makes a strong thesis and solicits input or discussion.

1. Acknowledge one specific point that resonates. Name it exactly, not generic praise. One sentence.
2. Add a sharp observation that extends or sharpens their point. One sentence, signals expertise, not just agreement.
3. Ask a targeted question about one specific capability gap. Give 2-3 concrete options to anchor the reply. No "I'm building" framing needed here.

### Step 4: Draft and Self-check

Apply tone rules below. Read the draft aloud. If it sounds like a template, rewrite it.

---

## Tone Rules

- No filler openers. No "great post!" or "interesting article". Name one specific thing from the post if acknowledging it, then pivot directly into the hook.
- Never hostile or dismissive. "Something worth adding" works better than "one gap this doesn't address".
- Do NOT pitch. Do NOT name the product. The bio handles that.
- No generic questions. "Has anyone solved this?" undermines credibility, the builder IS solving it.
- No em dashes. Use periods, commas, or colons instead.
- Must pass the senior engineer test: would a staff-level engineer write this sentence?
- LinkedIn: direct is acceptable. Mentioning you're building something reads as context, not marketing.

---

## Conversion Path

```
Comment adds value
  -> They reply or engage
  -> DM: "I'm building exactly this. 15 minutes to tell me about your setup? No pitch."
  -> Discovery call
  -> Pilot offer if pain is real
```

Do not DM without a public reply that got engagement first.

---

## What NOT to Do

- Comment with something the post already covers
- Use generic compliments even to soften an opener
- List multiple features, one hook per comment
- Ask "has anyone solved this?"
- Mention the product name or link in the first comment
- Edit AI drafts without reading aloud
- DM without a public reply that got engagement first

---

## Self-Improvement Protocol

This skill tracks corrections and improvements over time. When the user corrects Claude's behavior while this skill is active, the skill writes those observations to an improvement plan file in the marketplace repository. These plans are later reviewed by the skill author to implement permanent changes to the skill.

### During Execution: Monitor for Corrections

While executing this skill's workflow, watch for signals that the user is correcting your behavior:

**Correction signals to watch for:**
- Direct corrections: "no", "don't do that", "stop", "wrong", "that's not right", "I said..."
- Redirection: "instead, do X", "I meant Y", "use Z approach"
- Repeated instructions: the user restating something they already said (indicates you missed it)
- Frustration indicators: "again", "I already told you", "as I said before"
- Preference expressions: "I prefer", "always do X", "never do Y"
- Approval of non-obvious approaches: "yes exactly", "perfect", "that's the right way" (for approaches that were judgment calls, not obvious from the instructions)

**What to capture for each correction:**
1. **What you did wrong** (or what non-obvious approach worked well)
2. **What the user wanted instead** (or confirmed as correct)
3. **Which workflow step it relates to**
4. **Why the correction matters** (impact on output quality)

### Writing Improvements

After the skill's workflow completes (or at natural breakpoints if the session is long), write or update the improvement plan file at:

```
/Applications/workspace/gen-ai-projects/utibes-plugins-marketplace/improvement-plans/social-response-writer.md
```

Use this format:

```markdown
# Improvement Plan: social-response-writer

Last updated: {date}
Total lessons: {count}

## Lessons Learned

### Lesson {N}: {short title}
- **Date:** {date}
- **Workflow Step:** {step name/number}
- **What happened:** {what you did that was corrected, or what approach was confirmed}
- **What to do instead:** {the correct behavior, or the confirmed approach to keep using}
- **Why:** {why this matters for output quality}

## Patterns to Watch For

{Summarize recurring themes from the lessons above. For example, if the user has corrected tone multiple times, note: "User wants responses that pass the senior engineer test strictly. Always re-read drafts through that lens before presenting."}

## Proposed Skill Changes

{If a lesson is clear and repeated enough to warrant a permanent change to the SKILL.md instructions, document the proposed change here. Include which section to modify and the suggested new wording.}

| # | Section | Current Behavior | Proposed Change | Based on Lessons |
|---|---------|-----------------|-----------------|------------------|
| 1 | {section} | {what the skill currently says} | {what it should say} | Lessons {N, M} |
```

### Rules

1. **Never modify SKILL.md directly.** All improvements go to the improvement plan file as proposals for the skill author.
2. **Be specific.** "User didn't like the output" is useless. "User wanted the observation to reference a specific technical detail from the post, not a general statement about the problem space" is actionable.
3. **Capture successes too.** If you made a judgment call and the user confirmed it was right, record that as a positive lesson so future sessions maintain that behavior.
4. **Deduplicate.** If a new correction matches an existing lesson, update the existing lesson's count or add context rather than creating a duplicate.
5. **Keep it concise.** Target under 50 lessons; if it grows beyond that, consolidate related lessons into patterns.
6. **Note conformance requirements on every proposed change.** When writing entries in the "Proposed Skill Changes" table, add a note that the change must keep SKILL.md under 500 lines (moving detail to references/ files if needed) and that side-effect skills must retain `disable-model-invocation: true` in their frontmatter.
