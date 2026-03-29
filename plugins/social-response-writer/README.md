# Social Response Writer

A Claude Code plugin that drafts high-signal LinkedIn and Twitter/X replies for an AI agent observability founder. Converts social posts about AI agent failures, debugging pain, or production monitoring into comments designed to start conversations that lead to design partner pilots.

## Features

- Three response styles: Observation-led (default), Curiosity-led, and Acknowledge-and-ask
- Eight differentiation feature hooks to choose from per comment
- Competitive context for LangSmith, Langfuse, Helicone, Braintrust, Datadog, Arize, and Fiddler
- Tone rules that pass the senior engineer test
- Built-in conversion path from comment to DM to discovery call to pilot

## Usage

Ask naturally: "write a reply to this post", "draft a comment", "respond to this post", or "write a social response".

Provide the post content (paste or link) and the skill will:
1. Read the full post and any linked article
2. Pick the single most relevant feature hook
3. Choose the best response style
4. Draft and self-check against tone rules

## Installation

```
/plugin install social-response-writer@utibes-plugins-marketplace
```
