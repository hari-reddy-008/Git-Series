# Capstone Evidence Pack

This folder contains the evidence collected while reproducing and exercising the three
reference systems for the capstone.

The evidence is organized by system. Each system folder contains the relevant test,
static-analysis, execution, and perturbation captures. The top-level documents provide
the environment details, perturbation record, and final reflection.

## Submission Layout

```text
capstone-submission/
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
│   └── perturbation-missing-source.txt
│
├── 02-mortgage-extraction/
│   ├── tests.txt
│   ├── mypy.txt
│   ├── ruff.txt
│   ├── replay-appraisal.txt
│   ├── replay-missing-bonus.txt
│   ├── replay-sum-mismatch.txt
│   └── perturbation-validator-mismatch.txt
│
└── 03-supply-chain/
    ├── tests.txt
    ├── mypy.txt
    ├── ruff.txt
    ├── offline-meridian.txt
    └── perturbation-logistics-timeout.txt
