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

Ran it 11x and it used 2% of weekly limit, so one inference request consumes about:

```
2% ÷ 11 = 0.1818%  (% of the weekly plan)
0.1818% ÷ 4 = 0.04545%  (% of the monthly plan)

Each inference consumes about 0.045% of the monthly plan.
```

> Units are percentages (of *weekly plan*)

#### Codex

> Subscription Plan ($20 plan)

Codex only reveals what % of your weekly limit is consumed, unlike Kiro, which reveals the number of consumed credits. The percentage it shows is rounded to the nearest percentage point (no decimals). Therefore, I must run this prompt a few times to more accurately determine what % of the weekly limit was used up.

> Using GPT 5.4

Ran it 5x and it used 2% of weekly limit, so one inference requests consumes about: 

```
2% ÷ 5 = 0.4%  (% of the weekly plan)
0.4% ÷ 4 = 0.1%  (% of the monthly plah)

Each inference consumes about 0.1% of the monthly plan.
```

#### Github Copilot

> Using Opus 4.6

$10 for 100 inference requests

#### Cost Analysis

Fundamentally, every inference request sent to the harness must go through API behind the scenes. Our goal here is to analyze the true cost that every subscription-based harness incurs and compare it to the plan cost to determine how much 

**Kiro**
In order to compute the cost of a Kiro credit, we must compare it to either Cline or Claude Code, which expose the raw API costs. Since Kiro's (large) context window is closer in size to Cline, we can use Cline as the baseline to derive the user surplus (defined as `value received in dollars / price paid in dollars`)

0.54 credits (Kiro) = $0.15 (Cline)

```
1 Kiro credit = $0.15 / 0.54 ≈ $0.278
```

$20 Kiro plan (1,000 credits):

```
1000 credits × $0.278/credit = $277.78 in true API cost
```

That's **~14x** the price of the plan!
 
**Claude Code**
The results above reveal that I get 2200x of these inference requests. Assuming that each inference request costs the same as what Claude Code's API costs (refer to README.m to understand why this assumption is valid), the true cost that Claude Code incurs can be computed as:

```
2200 x $0.067 = $147.4 in true API cost
```

That's ~7.4x the price of the plan.

**Codex**
The results above reveal that I get 1000x of these inference requests. Assuming that each inference request costs the same as what Claude Code's API costs (refer to README.md to understand why this assumption is valid), the true cost that Codex incurs can be computed as:

> We compare to Claude Code API cost and not Cline's, as Codex's system prompt is presumably more similar to Claude Code, both being from frontier model companies. This is a guess.

```
1000 × 0.067 = $67 in true API cost
```

That's ~3.5x the price of the plan.

**Github Copilot**

```
100 * 0.067 = $6.7 in true API cost
```

That's 0.67x the price of thep lan

### Scenario 2: Explaining Code

*TODO*

### Scenario 3: Writing Code

*TODO*


## Conclusions

| Harness | Value Leverage | Scenario |
|---|---:|---|
| Kiro | 14 | Explaining a Concept |
| Claude Code | 7.4 | Explaining a Concept |
| Codex | 3.5 | Explaining a Concept |
| GitHub Copilot | 0.67 | Explaining a Concept |

> 'Value Leverage' is defined as the monetary value of inference that consumers of the harness receive relative to the price they pay

- Claude Code is cheaper than Cline despite using both using the same model and having no markups. This is most likely due to Cline's longer system prompts, which was verified by seeing token consumption. This doesn't mean Claude Code is inherently "better" though; longer system prompts could lead to better task performance (which is probably the reason for longer system prompts)

- Despite popular belief and empirical evidence, Claude Code seems about two times cheaper than Codex. This is most likely due to Claude Code being cheaper for explaining concepts; I'm pretty sure that Claude Code will be more expensive than Codex for coding tasks

- Obviously, Copilot is not worth it for simple requests. So it's expected that Copilot will rank better for the more difficult scenarios