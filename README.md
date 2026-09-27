# Capstone Evidence Pack

The evidence which was gathered during the process of reproducing and testing the three reference systems is contained in this folder.

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
```
## System 1 — Validated, Routed Insurance Policy Extraction

The evidence consists of the entire test suite, the results of the type-checking and linting, the routing tests, the calibration evidence, and the results of the controlled perturbation involving a missing source.

The test suite as a whole resulted in 45 tests passing and 3 being skipped; the ones that were skipped were the live Anthropic API tests since there was no ANTHROPIC_API_KEY available.

All nine routing tests were passed in the offline routing test. The live API pipeline was not run without credentials, and no live API-generated routing artifact was created.

## System 2 — Resilient Mortgage Document Extraction

The evidence includes the full test suite, results from static analysis, offline replay captures, and a controlled validator change.

The entire test suite resulted in 25 tests passing.

The replay shows a failure to display the appraisal, the unavailable bonus information, and a discrepancy in the monthly income figure. The case involving the discrepancy illustrates the explicit identification of an inconsistency rather than simply accepting the given total.

## System 3 — Supply Chain Risk Investigation

The evidence consists of the entire test suite, the static-analysis results, an offline investigation into Meridian, and a simulation of a logistics-timeout scenario.

The entire test suite resulted in 34 tests passing.

The evidence from the investigation keeps track of the source's origin and identifies findings that have been corroborated, those based on a single source, contested ones, and those which are incomplete. The result obtained after the timeout period indicates that the investigation carries on by recording the unavailable logistics source as an incomplete coverage area.

The fact that the mypy limitation for System 3 is recorded in environment.txt and the relevant evidence file rather than being indicated as a successful type check.

## Perturbation Evidence

One deliberate perturbation was performed for each system:

Policy pipeline: missing endorsements source.
- In the case of a mortgage extraction, the stated total exceeds the validator's tolerance.
- Supply chain: a simulation of a logistics timeout.

The observations are recorded in the file perturbation-log.md.

## Reflection

The file called reflection-brief.md includes the final reflection and relates the observations set out in it to the evidence gathered during the execution.

The results that are reported here are based on the actual executions included in this pack and not on any live API results which were unavailable.
