# AI Agent Harness Cost Benchmarking

This project benchmarks AI coding agent harnesses on cost alone — not response quality or latency.

We assume that response quality is effectively constant across harnesses. I will only ever use SOTA models, and these all largely converge.

## Agent Harnesses

The agent harnesses being evaluated are:

- **Amazon Kiro**

Uses the concept of "credits", which is basically an abstraction layer over tokens, for usage metrics and pricing. We need to be able to convert "credits" to dollars in order to compare it to other harnesses.

- **Cline**
- **Claude Code**
- **Codex**