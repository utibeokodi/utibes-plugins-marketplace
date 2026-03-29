# Canonical Example Responses

## Example 1: Style A (Observation-led)

**Post type:** Thought leadership about why production monitoring for AI agents is different from traditional observability.

**Hook:** Automated RCA

**Response:**
> Something worth adding to this: automated root cause analysis.
>
> Agents generate traces fast. During a production incident, manually sifting through thousands of traces is exhausting. Dumping them all into an LLM to find the root cause gets expensive quickly.
>
> This is exactly what I'm building toward. An observability platform where RCA is automatic, not a manual excavation job.

**Why it works:**
- Extends the post without repeating it
- Names a real developer pain: trace volume and LLM cost
- Founder framing is credible on LinkedIn
- No product name, no pitch, no question

---

## Example 2: Style B (Curiosity-led)

**Post type:** Team describing an internal AI agent system they built, moving toward full autonomy.

**Hook:** Behavioral Anomaly Detection

**Response:**
> Curious how you're debugging failures when the agent gets it wrong. Especially as you move toward full autonomy.
>
> At that level of agent involvement, failures stop being obvious. They're subtle: a tool call loop that completes, an output that looks right but isn't, a diagnosis that's confidently wrong. By the time you notice, the trace is buried.
>
> That's the exact problem I'm building toward solving. Making those failures visible automatically, before you have to go looking.

**Why it works:**
- Opens with a specific, genuine question tied to their setup (not generic)
- Names the pain they'll hit as autonomy increases: subtle failures, not crashes
- Founder framing is credible without pitching
- No product name, no list of features

---

## Example 3: Style C (Acknowledge and ask)

**Post type:** Thought leadership about rebuilding the agent infrastructure stack from first principles, soliciting input on what to prioritize.

**Hook:** Automated RCA / Observability

**Response:**
> Agreed, especially on observability. Debugging million-path execution trees with today's tools is like reading a core dump with no symbols. What's the one capability you wish existed for agent execution: root cause surfaced automatically, execution replay, or something else entirely?

**Why it works:**
- Acknowledges a specific point (observability), not the post in general
- The analogy ("core dump with no symbols") adds a sharp, credible observation before the question
- Question anchors replies with concrete options, making it easy to respond
- No "I'm building" framing needed, the question itself signals deep domain knowledge
- Short enough for Twitter, strong enough for LinkedIn

---

## Pattern to internalize

Each example does exactly three things:
1. Adds something the original post didn't say
2. Names a pain that is real and specific to developers at scale
3. Signals credibility without naming the product or pitching

The conversion happens in the DM, not the comment.
