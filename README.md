# Prompt Maker

A Claude Skill that transforms weak or unstructured prompts into high-performance structured prompts, with a quality score and a clear iteration path on every output.

Version 7.1, updated June 2026 against Anthropic's official prompting guides for Claude Fable 5 and Claude Opus 4.8.

## What it does

Give it a rough prompt or just describe a task, and it returns an optimized prompt that is:

- **Model aware.** Four routes built from each vendor's official prompting guidance: GPT-5.5 (Route A), Claude Fable 5 (Route B), Claude Opus 4.8 (Route C), and Claude Sonnet 4.6 (Route D). Each route encodes the model's real behavioral differences: scaffolding style, effort calibration, instruction-following quirks, pricing, and known migration traps.
- **Complexity tiered.** Simple, Standard, and Complex tiers control how much scaffolding gets applied. A 50-word reply does not get a self-check block; a multi-day agentic prompt gets the full Long-Run Pack.
- **Personalization aware.** Hardcodes the known user's identity and documented preferences instead of leaving lazy brackets, and flags what to anonymize before a prompt is shared.
- **Scored.** Every delivery includes a 1 to 100 quality score across 10 dimensions, with the two weakest dimensions called out and a fix suggested.
- **Iterable.** Fifteen named iteration protocols (A through O) map common failure patterns to surgical fixes, so a bad first run rarely needs a full rewrite.

## Highlights of the model routes

- **Claude Fable 5:** intent-first scaffolding (prescriptive steps degrade output), effort defaults, the Long-Run Pack for autonomous agents (progress grounding, boundaries, subagents, memory), and classifier fallback awareness.
- **Claude Opus 4.8:** literal instruction following, five effort levels, coverage-first code review prompting, and how to escape the model's default design house style.
- **Claude Sonnet 4.6:** the volume production route, including the prefill deprecation and the budget_tokens to effort migration.
- **GPT-5.5:** outcome-first, process-light prompting with explicit success criteria.

A Claude Family Selector and a workload-to-model matrix pick the right default when you have not chosen a model yourself.

## Install

**Claude.ai / Claude apps:** Settings → Capabilities → Skills → upload `prompt-maker.skill`.

**Claude Code:** copy the `prompt-maker/` folder into your skills directory.

## Usage

Trigger it with phrases like "improve this prompt", "create a prompt for X", "apply the prompt maker method", or just paste a weak prompt and ask for enhancement.

## Credits

Built on the Prompt Maker Method by Ruben Hassid, extended through v7.1 with model-aware routing, complexity tiers, a quality rubric, and iteration protocols. Model-specific guidance sourced from Anthropic's official prompting documentation (June 2026).

## License

MIT. See [LICENSE](LICENSE).
