# Reflection Brief

## 1. System 1 — The extraction of insurance policies that have been validated and routed

The execution path in question showed that the test suite had passed and that the static checks were clear: 45 of the tests passed, while the three live API tests were skipped since the ANTHROPIC_API_KEY was not available. All nine of the routing tests passed for the documented routing fallback. The routing tests included cases for automatic approval of eligible high-confidence results, human review in the case of low confidence and integration failures, reviewer disagreement, stratified spot checks, and calibration behaviour.

The calibration report revealed a significant risk in that an overall result can conceal information—the general Brier score was 0.291, whereas the 'umbrella/exclusions' section had n=2, a confidence level of 0.93, an accuracy of 0.00 and a Brier score of 0.865. Therefore, it is advisable to use sliced calibration evidence rather than depending solely on the aggregate calibration figure.

The controlled missing-source perturbation was also successful; it triggered the `endorsements`-absent path and confirmed that immediate escalation took place rather than repeated futile retries.

The Anthropic pipeline was not run since the necessary API key was not available, and therefore no assertion can be made regarding a live-generated routing_decisions.json artifact.

## 2. System 2, Resilient Mortgage Document Extraction

The entire test suite achieved 25 out of 25 with clean reports from both mypy and Ruff; the offline replay scenarios showed three different behaviours.

The typed extraction from the appraisal replay showed the gross_living_area_sqft value set at 2400 and included a valid result with no discrepancies. In the case of the missing-bonus replay, the unavailable bonus information was given a value of null and yet remained consistent. The income-sum-mismatch replay found an inconsistency since the calculated monthly income was 9642.17 whereas the stated amount was 10892.17, the difference being -1250.00.

The validator's controlled perturbation was also accepted. Taken together, these artifacts demonstrate the advantage of including both calculated and stated values and of making mathematical discrepancies explicit rather than simply accepting a stated total.

## 3. System Three — Investigation into Supply Chain Risk

The full test suite passed all 34 of its tests and Ruff also passed. The offline Meridian run resulted in a briefing which maintained provenance and was able to distinguish between findings that were corroborated, those based on a single source, contested ones, and those that were incomplete.

For instance, the figure for `average_lead_time_days` was confirmed by two different sources and stated to be 12.0 days, whereas `on_time_delivery_rate` was directly disputed with 95.0 percent given in the supplier_audit and 78.0 percent cited by logistics. Additionally, the briefing raised an ambiguous claim regarding the supplier's identity and also highlighted an incomplete high-impact metric for which there was no reliable source.

The logistics-timeout perturbation was resolved when the project's `.venv` was run. The coordinator then carried on with the investigation and noted the coverage gap that resulted, rather than aborting. This shows the difference between a source failure and a valid empty result, as well as the importance of keeping the failure context when continuing with the available sources.

The project check failed when using the mypy command for System 3 since the mypy configuration provided targets Python 3.11 but the installed NumPy stub contains syntax that requires Python 3.12 or later. The fact that this limitation exists is recorded as evidence rather than treating it as a successful type check.

## Cross-system reflection

In all three systems the most frequent design feature was the explicit management of uncertainty and failure. The evidence reveals different methods of achieving this: human-review routing and sliced calibration in the policy pipeline, the direct reporting of mathematical discrepancies in mortgage extraction, and provenance-aware partial results, contested claims, and coverage gaps in the supply-chain investigation.

The disturbances were useful since they allowed the behaviour of the system to be tested under adverse conditions rather than just its normal, straightforward functionality. In every instance, the test or fixture provided expected the system to make the problem apparent rather than conceal it by, for example, escalating a missing policy source, flagging an inconsistent income total, or proceeding despite a documented source failure.

One drawback of this evidence pack is that System 1's live Anthropic end-to-end pipeline could not be run without an API key, and the evidence therefore makes use of the project's documented routing-test fallback in that case. Similarly, System 3 also has the documented mypy environment/configuration limitation mentioned above.
