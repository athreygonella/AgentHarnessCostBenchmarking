# AI Agent Harness Cost Benchmarking

This project benchmarks AI coding agent harnesses on cost alone — not response quality or latency.

We assume that response quality is effectively constant across harnesses. I will only ever use SOTA models, and these all largely converge.

## Agent Harnesses

The agent harnesses being evaluated are:

#### Amazon Kiro

Uses the concept of "credits", which is basically an abstraction layer over tokens, for usage metrics and pricing. We need to be able to convert "credits" to dollars in order to compare it to other harnesses.

#### Cline
#### Claude Code
#### Codex
#### Github Copilot

Has a different pricing structure compared to other harnesses. For a $10 plan, you have a flat 100 Opus 4.6 requests (or 300 GPT 5.4 requests). This means that your price leverage depends on how you use those requests - whether you use it for simple questions or complex, long tasks. This could be very interesting and useful, especially if paired with a different harness used exlusively for simple questions and prompt-building, which then gets executed by the copilot harness.