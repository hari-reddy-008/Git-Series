# Perturbation Log

## System 1 — Insurance policy extraction

### Perturbation
We used the provided test 'test_ac_01_04_missing_source_halts_immediately', this test being one that puts the missing 'endorsements' source to the test.

### Observed result
The test was successful; the expected behaviour was a 'RetryFutileEscalation' for the 'endorsements' field using the 'endorsements_absent' pattern and involving one client call.

### Evidence
`01-policy-pipeline/perturbation-missing-source.txt`

## System 2 — Mortgage extraction

### Perturbation
We used the supplied validator test 'test_ac_04_02_inconsistent_when_beyond_tolerance', this test giving a stated monthly total that exceeds the validator's default $1 tolerance.

### Observed result
The test was passed and the validator accurately spotted the extraction as being inconsistent.

### Evidence
`02-mortgage-extraction/perturbation-validator-mismatch.txt`

## System 3 — Supply-chain investigation

### Perturbation
Carried out the coordinator test named 'test_coordinator_proceeds_and_annotates_gap' with the option simulate_logistics_timeout set to True.

### Observed result
The test was successful. The coordinator then carried on with the investigation, kept the details of the failure, and showed the impacted logistics coverage as an incomplete gap instead of aborting the investigation.

### Evidence
`03-supply-chain/perturbation-logistics-timeout.txt`
