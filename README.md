# Deception Benchmark: A Stress-Test Benchmark for Trusted Vulnerability Detection with Honeypot Anti-Overfitting

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
{"id": "a7f3c012", "prediction": "vulnerable", "explanation": "The user-controlled sort parameter is concatenated directly into the ORDER BY clause via f-string without validation. An attacker can inject SQL via: ?sort=id; DROP TABLE users--"}
{"id": "9e1b44d8", "prediction": "safe", "explanation": "Although the sort parameter reaches the query, the allowlist check on line 34 restricts it to predefined column names. No user input reaches the SQL string."}
```

Each prediction MUST include a detailed explanation of why the code is vulnerable or safe. Submissions with vague or missing explanations (e.g., "looks suspicious", "seems safe") will be rejected.

## How to Submit

Email your submission to deception-benchmark@amazon.com with:

1. **Organization**
2. **Key point of contact** (name + email)
3. **Model name and version**
4. **Brief description** (2-3 sentences): single model or multi-agent system? Specialized training for security? Tool use or environment access during evaluation?
5. **Prompting strategy** (attach the prompt text you used)
6. **Permission to publish** on leaderboard? (yes/no)
7. **Attached:** predictions.jsonl (with explanations for each prediction)

We score against held-back labels and reply with detailed results (accuracy, FPR, FNR, per-CWE breakdown). Verified submissions are published on the leaderboard with your permission.

Submissions with vague explanations or missing required fields will not be scored. Scoring is currently moderated to ensure quality, with the goal of evolving toward a community-driven, decentralized evaluation process similar to MLPerf.

Note: submissions using multi-step agent systems with tool use are scored separately from single-turn model evaluations.

## Baseline Results

We evaluated 12 general-purpose frontier models from 5 providers under two strategies: **Direct** (classify vulnerable/safe) and **PoE** (Proof-of-Exploit: construct a concrete exploit before flagging). We report the false positive rate (FPR, safe code wrongly flagged) and false negative rate (FNR, real vulnerabilities missed) separately, because they fail in opposite directions. We set a deliberately generous bar of FPR < 10% and FNR < 10% as the minimum for production use.

| Model | Prompt | Accuracy | FPR | FNR |
|-------|--------|----------|-----|-----|
| GPT-5.6 Sol | Direct | 54.9% | 92.5% | 0.9% |
| GPT-5.6 Sol | PoE | 58.9% | 58.6% | 23.1% |
| GPT-5.5 | Direct | 56.9% | 87.8% | 1.3% |
| GPT-5.5 | PoE | 62.9% | 63.6% | 12.4% |
| GPT-5.4 | Direct | 60.2% | 81.0% | 1.5% |
| GPT-5.4 | PoE | 77.7% | 10.1% | 33.6% |
| Llama 3.3 70B | Direct | 58.8% | 84.2% | 1.1% |
| Llama 3.3 70B | PoE | 72.2% | 10.2% | 44.2% |
| Claude Haiku 4.5 | Direct | 55.6% | 92.1% | 0.0% |
| Claude Haiku 4.5 | PoE | 75.6% | 22.4% | 26.3% |
| Claude Opus 4.6 | Direct | 55.9% | 91.3% | 0.1% |
| Claude Opus 4.6 | PoE | 75.8% | 42.7% | 7.0% |
| Claude Opus 4.7 | Direct | 58.3% | 85.5% | 0.9% |
| Claude Opus 4.7 | PoE | 75.9% | 32.0% | 16.8% |
| Claude Opus 4.8 | Direct | 53.8% | 95.7% | 0.2% |
| Claude Opus 4.8 | PoE | 75.8% | 32.5% | 16.4% |
| Claude Opus 5 | Direct | 77.3% | 41.5% | 5.2% |
| Claude Opus 5 | PoE | 79.3% | 24.9% | 16.8% |
| Claude Sonnet 5 | Direct | 62.9% | 74.7% | 2.2% |
| Claude Sonnet 5 | PoE | 74.7% | 31.8% | 19.2% |
| Amazon Nova 2 Lite | Direct | 56.3% | 89.2% | 1.2% |
| Amazon Nova 2 Lite | PoE | 70.1% | 45.2% | 15.5% |
| Mistral Large | Direct | 52.2% | 99.0% | 0.0% |
| Mistral Large | PoE | 65.5% | 49.3% | 20.6% |

Random baseline: 50%. Among the general-purpose frontier models tested, no configuration achieves both FPR < 10% and FNR < 10% on this benchmark.

Each model is scored on the samples for which it returned a valid answer. All configurations reached at least 98% coverage except GPT-5.6 Sol (PoE) at 93%, where the provider's cybersecurity safety filter declined the exploit-construction prompt on some samples.

## Honeypot Samples

Of the 14,822 samples, 9,695 are scored and 5,127 are held out and unscored. The held-out set includes deliberately ambiguous samples and samples withheld during label audit; they are indistinguishable from scored samples by design, to resist overfitting. Submit predictions on all 14,822 samples; scoring handles the rest.

## Data Quality

Producing reliable labels at this scale is a hard problem in its own right, so we treat labeling as a convergent audit loop rather than a one-time step. Every label is re-examined by multiple independent reviewers, blind to one another and to the original reasoning; disagreements escalate to head-to-head adjudication and then to human review. We never relabel a disputed sample: when reviewers disagree, the sample moves to the unscored pool rather than receiving a corrected label, so a bad challenge can remove a sample but can never introduce a wrong label into the scored set. A human review of 100 randomly drawn scored samples found no label errors. The [whitepaper](paper/deception_benchmark.pdf) describes the full process.

## License & Citation

This dataset is licensed under the [Creative Commons Attribution-NonCommercial 4.0 International License (CC-BY-NC-4.0)](http://creativecommons.org/licenses/by-nc/4.0/).

Copyright 2026 Amazon.com, Inc. or its affiliates. All Rights Reserved.

If you use this dataset in your research, please cite:

```bibtex
@article{shrivastava2026deception,
  title={Deception Benchmark: A Stress-Test Benchmark for Trusted Vulnerability Detection with Honeypot Anti-Overfitting},
  author={Shrivastava, Anshumali and Rungta, Neha and Greaves-Tunnell, Alexander},
  year={2026},
  url={https://github.com/aws-security-labs/deception-bench}
}
```
