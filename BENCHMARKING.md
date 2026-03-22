# Benchmarking

## Methodology

Cost benchmarking relies on controlled, fixed prompts — the prompt is the constant across all harnesses. This benchmarking exercise will use a few different prompts that span different task types (`explain a concept`, `explain code`, `write code`) to identify if and how cost profiles vary based on whether the model is generating prose vs. code vs. reasoning through existing code.

### Controlling for Bias

Several design choices are made to reduce confounding factors:

- **Single-turn only.** Only the first prompt in a fresh session is measured. Multi-turn conversations create potential for increased distortion / complexity, and also may accumulate context that inflates token counts unpredictably, making cost comparisons unreliable.

- **Identical prompts across harnesses.** The exact same prompt text is used for every harness in a given scenario. This ensures differences in cost stem from the harness itself (system prompt size, routing, tokenization) rather than from variation in what was asked.

- **Multiple runs per harness.** Each scenario is executed five times to smooth out per-run variations due to caching behavior, warm-up effects, or other non-deterministic cost fluctuations. Run 1 is treated as the canonical "cold start" cost; subsequent runs reveal whether the harness applies prompt caching or other optimizations.

- **Detailed, specific prompts** Because each harness ships its own system prompt, in order to mitigate differences in token consumption and reasoning that arise due to the system prompt, the mitigation strategy (on a hunch) is to provide a very detailed, specific prompt that tells the agent exactly what to do, hopefully reducing reliance on the system prompt.

## Scenarios

### Scenario 1: Explaining a Concept

Prompt: `Explain event sourcing. Start with an introduction / motivation (what problem is it solving?), fundamental explanation of what it is and how it works, and finally a specific, detailed, but brief example - implemented in TS`

#### Kiro

> Using Opus 4.6

| Run | 1    | 2    | 3    | 4    | 5    |
|-----|------|------|------|------|------|
| IDE | **0.54** | 0.38 | 0.38 | 0.38 | 0.41 |
| CLI | **0.53** | 0.36 | 0.37 | 0.38 | 0.39 |

> Units are Kiro credits

> Token usage was ~64k (inferred from the percentage of context window - 32% - that's surfaced)

Results suggest some form of caching after the first inference — cost drops noticeably from run 1 to run 2, then plateaus. This holds across both IDE and CLI, so going forward we'll focus on first-request costs only for Kiro.

#### Cline

> Using Opus 4.6

| Run | 1    | 2     | 3     | 4     | 5     |
|-----|------|-------|-------|-------|-------|
|     | **0.15** | 0.066 | 0.063 | 0.057 | 0.053 |

> Units are dollars

> Token usage was 19k (incl. system prompt + inference)

#### Claude Code

> Using Opus 4.6

**API Pricing**

> Claude Code via AWS Bedrock (same as using it via Anthropic; AWS doesn't add a markup)

| Run | 1         | 2     | 3     | 4     | 5     |
|-----|-----------|-------|-------|-------|-------|
|     | **0.067** | 0.040 | 0.051 | 0.039 | 0.043 |

> Units are dollars

> Token usage was 5k (incl. system prompt + inference)

**Subscription Plan**

TODO: I need to figure out what % of a weekly plan this query consumes. Then, I can determine how much this costs in a month, relative to the fixed price of $100 or whatever. and can also then compare it to the api pricing. and compare it to kiro to see the "true cost".

| Run | 1 | 2 | 3 | 4 | 5 |
|-----|---|---|---|---|---|
|     |   |   |   |   |   |



> Units are percentages (of *weekly plan*)

#### Codex

> Using GPT 5.4

TODO

#### Cost Analysis

(incomplete - todo)

Fundamentally, everything goes through API in the harness backend. Both Cline and Claude Code expose raw API costs directly, but since Kiro's (large!) context window is closer to Cline's, we'll use Cline as the baseline to derive the true dollar value of a Kiro credit."

- Kiro: 0.54 credits
- Cline: $0.15

```
1 Kiro credit = $0.15 / 0.54 ≈ $0.278
```

$20 Kiro plan (1,000 credits):

```
1000 credits × $0.278/credit = $277.78 in true API cost
```

That's ~14x the plan price in actual API spend.


### Scenario 2: Explaining Code

*TODO*

### Scenario 3: Writing Code

*TODO*


## Conclusions

- Claude Code is cheaper than Cline despite using both using the same model and having no markups. This is most likely due to Cline's longer system prompts, which was verified by seeing token consumption. This doesn't mean Claude Code is inherently "better" though - these longer system prompts could lead to better performance on tasks (that is probably the intent of the longer system prompt)

- 