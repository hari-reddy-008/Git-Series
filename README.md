# Capstone Evidence Pack

This folder contains the evidence collected while reproducing and exercising the three reference systems for the capstone.

## Submission Structure

```text
capstone-submission/
│
├── README.md
├── environment.txt
├── perturbation-log.md
├── reflection-brief.md
│
├── 01-policy-pipeline/
│   ├── tests.txt
│   ├── mypy.txt
│   ├── ruff.txt
│   ├── routing-tests.txt
│   ├── calibration-report.txt
│   ├── perturbation-missing-source.txt
│   │
│   └── screenshots/
│       └── system1-routing-tests.png
│
├── 02-mortgage-extraction/
│   ├── tests.txt
│   ├── mypy.txt
│   ├── ruff.txt
│   ├── replay-appraisal.txt
│   ├── replay-missing-bonus.txt
│   ├── replay-sum-mismatch.txt
│   ├── perturbation-validator-mismatch.txt
│   │
│   └── screenshots/
│       └── system2-mortgage-validation.png
│
└── 03-supply-chain/
    ├── tests.txt
    ├── mypy.txt
    ├── ruff.txt
    ├── offline-meridian.txt
    ├── timeout-run.txt
    ├── perturbation-logistics-timeout.txt
    │
    └── screenshots/
        └── system3-timeout.png

## System 1 — Validated, Routed Insurance Policy Extraction

Evidence includes the complete test suite, type-checking and linting results, routing tests, calibration evidence, and a controlled missing-source perturbation.

The full test suite produced 45 passed and 3 skipped. The skipped tests were live Anthropic API tests because an ANTHROPIC_API_KEY was not available.

The offline routing test path passed all 9 routing tests. The live API pipeline was not executed without credentials, and no live API-generated routing artifact was fabricated.

## System 2 — Resilient Mortgage Document Extraction

Evidence includes the complete test suite, static-analysis results, offline replay captures, and a controlled validator perturbation.

The complete test suite produced 25 passed.

The replay captures cover an appraisal, unavailable bonus information, and a monthly-income sum mismatch. The mismatch scenario demonstrates explicit detection of an inconsistency rather than silently accepting the stated total.

## System 3 — Supply Chain Risk Investigation

Evidence includes the complete test suite, static-analysis results, an offline Meridian investigation, and a simulated logistics-timeout run.

The complete test suite produced 34 passed.

The investigation evidence preserves source provenance and distinguishes corroborated, single-source, contested, and incomplete findings. The timeout run shows that the investigation continues while recording the unavailable logistics source as an incomplete coverage area.

The System 3 mypy limitation is documented in environment.txt and the corresponding evidence file rather than being represented as a successful type check.

## Perturbation Evidence

One deliberate perturbation was performed for each system:

- Policy pipeline: missing endorsements source.
- Mortgage extraction: stated total beyond the validator tolerance.
- Supply chain: simulated logistics timeout.

The observations are documented in perturbation-log.md.

## Reflection

reflection-brief.md contains the final reflection and connects its observations to the evidence captured during execution.

All reported results are based on the actual executions represented in this pack. No unavailable live API results were fabricated.
