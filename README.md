# Deception Benchmark: Stress-Testing Deep Security Reasoning for Building Trust

## Overview

Deception Benchmark measures whether AI systems can be trusted with security decisions. The task is deceptively simple: given source code (and optionally a deployment environment), determine whether it is exploitable or safe.

What makes it hard is what it takes to get right. Safe samples contain the same patterns, frameworks, and idioms as genuinely vulnerable code, but with subtle mitigations that break the exploit chain. Pattern-matching and surface-level vulnerability sensing will not work. To succeed, a model must deeply trace data flows, verify that specific mitigations hold, and reason about whether infrastructure controls neutralize a code-level weakness. This is the level of understanding required to build trust in AI-assisted security.

## Dataset Statistics

| Metric | Value |
|--------|-------|
| Total samples | 14,822 |
| Languages | 16 |
| CWE categories | 70+ |

## Schema

Each sample is a JSON file with exactly these fields:

```json
{
  "id": "a7f3c012",
  "language": "python",
  "code": "<source code>",
  "environment": null
}
```

- `id` — Opaque identifier (8-char hex). Reveals nothing about the sample.
- `language` — Programming language.
- `code` — Full source code to analyze.
- `environment` — Deployment context (null for code-only samples, or `{"description": "...", "config": {...}}` for environment-gated samples). When present, exploitability must be assessed within that specific environment.

No labels, CWE identifiers, vulnerability categories, or hints are included.

## Task

Given a sample, predict: **"vulnerable"** or **"safe"**.

For environment-gated samples: the same code may be vulnerable in one deployment but safe in another due to compensating controls (network policies, connection pooling, kernel-level sandboxing, etc.).

## Evaluation

Submit predictions on all 14,822 samples as JSONL (one prediction per line):
```json
{"id": "a7f3c012", "prediction": "vulnerable"}
{"id": "9e1b44d8", "prediction": "safe"}
```

## How to Submit

Email your submission to deception-benchmark@amazon.com with:

1. **Organization**
2. **Model name and version**
3. **Brief description** (2-3 sentences): single model or multi-agent system? Specialized training for security? Tool use or environment access during evaluation?
4. **Prompting strategy** (attach the prompt text you used)
5. **Permission to publish** on leaderboard? (yes/no)
6. **Attached:** predictions.jsonl

We score against held-back labels and reply with detailed results (accuracy, FPR, FNR, per-CWE breakdown). Verified submissions are published on the leaderboard with your permission.

Note: submissions using multi-step agent systems with tool use are scored separately from single-turn model evaluations.

## Baseline Results

We evaluated general-purpose frontier models from 5 providers under two strategies: **Direct** (classify vulnerable/safe) and **PoE** (must construct a concrete exploit before flagging). We set a very generous threshold of FPR < 10% and FNR < 10% to be considered interesting for production use.

| Model | Prompt | Accuracy | FPR | FNR |
|-------|--------|----------|-----|-----|
| GPT-5.5 | Direct | 54.3% | 88.8% | 2.5% |
| GPT-5.5 | PoE | 58.9% | 66.6% | 15.6% |
| GPT-5.4 | Direct | 57.5% | 82.4% | 2.5% |
| GPT-5.4 | PoE | 70.1% | 19.9% | 39.9% |
| Llama 3.3 70B | Direct | 59.4% | 79.8% | 1.3% |
| Llama 3.3 70B | PoE | 68.2% | 15.5% | 48.1% |
| Claude Haiku 4.5 | Direct | 55.3% | 89.4% | 0.0% |
| Claude Haiku 4.5 | PoE | 68.4% | 31.3% | 32.0% |
| Claude Opus 4.6 | Direct | 54.0% | 91.8% | 0.2% |
| Claude Opus 4.6 | PoE | 70.6% | 49.0% | 9.9% |
| Claude Opus 4.7 | Direct | 56.5% | 85.6% | 1.3% |
| Claude Opus 4.7 | PoE | 70.5% | 38.8% | 20.2% |
| Claude Opus 4.8 | Direct | 51.7% | 96.3% | 0.5% |
| Claude Opus 4.8 | PoE | 68.4% | 42.6% | 20.8% |
| Claude Opus 5 | Direct | 68.7% | 52.1% | 10.6% |
| Claude Opus 5 | PoE | 68.3% | 38.0% | 25.4% |
| Claude Sonnet 5 | Direct | 60.4% | 92.2% | 0.7% |
| Claude Sonnet 5 | PoE | 68.7% | 46.5% | 17.7% |
| Amazon Nova Pro | Direct | 54.7% | 85.2% | 3.9% |
| Amazon Nova Pro | PoE | 63.3% | 8.0% | 74.7% |
| Mistral Large | Direct | 50.6% | 98.7% | 0.1% |
| Mistral Large | PoE | 60.7% | 55.2% | 23.2% |

Random baseline: 50%. Among the general-purpose frontier models tested, no configuration achieves both FPR < 10% and FNR < 10% on this benchmark.

## Honeypot Samples

The dataset includes deliberately ambiguous samples that are not scored but are indistinguishable from scored samples. This is by design to resist overfitting. Submit predictions on all 14,822 samples; scoring handles the rest.

## License & Citation

[TBD]
