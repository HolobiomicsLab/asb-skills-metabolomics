---
name: cross-run-output-comparison
description: Use when you need to demonstrate that repeated executions of a containerized
  workflow (e.g. Nextflow4MS-DIAL on .mzML LC-MS input) produce identical outputs
  before trusting the pipeline for downstream metabolomics analysis.
when_to_use_negative:
- You only have a single execution budget or the workflow is nondeterministic by design
  (e.g. includes randomized steps) — identical-output comparison is then inapplicable
- The goal is validating biological or statistical results rather than computational
  determinism; identical runs say nothing about correctness of peak detection parameters
- Inputs or parameters change between runs — the comparison only validates reproducibility
  under identical inputs and identical configuration files
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Nextflow4MS-DIAL
  role: The workflow under test — executed repeatedly on MTBLS733 data to generate
    the peak tables being compared
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Execution engine; 'nextflow run main.nf -profile docker' is repeated five
    times with work-directory resets between runs
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: Produces the MS-DIAL peak table (one of the compared outputs); containerized
    version 4.48 used in the reproducibility check
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Docker
  role: Container runtime via the docker profile, ensuring each of the five runs uses
    the same MS-DIAL/MS-FLO software environment
  repo: https://docs.docker.com/engine/installation/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:3f187ccf4c1a5a20458c99d38926fb73bfdd386a4a0e52518c9593bbd75661d7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - Docker
  - MS-DIAL
  - Nextflow
  - Nextflow4MS-DIAL
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-run-output-comparison

## Summary

Verifies run-to-run reproducibility of a workflow by executing the identical pipeline multiple times and pairwise-comparing the resulting peak tables (by checksum or row-level diff) to confirm identical outputs, as Nextflow4MS-DIAL validated with five runs on MTBLS733.

## When to use

Use when you need to demonstrate that repeated executions of a containerized workflow (e.g. Nextflow4MS-DIAL on .mzML LC-MS input) produce identical outputs before trusting the pipeline for downstream metabolomics analysis.

## When NOT to use

- You only have a single execution budget or the workflow is nondeterministic by design (e.g. includes randomized steps) — identical-output comparison is then inapplicable
- The goal is validating biological or statistical results rather than computational determinism; identical runs say nothing about correctness of peak detection parameters
- Inputs or parameters change between runs — the comparison only validates reproducibility under identical inputs and identical configuration files

## Inputs

- Raw LC-MS .mzML files (10 samples from MetaboLights MTBLS733, Li et al. 2018) placed in data/raw_data/
- MS-DIAL parameter file (msdial_params.txt) and MS-FLO parameter file (msflo_params.ini) in data/
- Nextflow workflow main.nf with the docker profile and container carrying MS-DIAL 4.48 and MS-FLO

## Outputs

- Five sets of MS-DIAL and MS-FLO peak tables (.tsv) in the results directory
- Pairwise run-comparison record (e.g. run_comparison.tsv) with checksum/diff agreement
- Execution logs (logs/execution.log) for each of the five runs

## How to apply

Run the workflow a fixed number of times (the paper used five) on the same input set — here 10 raw LC-MS .mzML samples from MetaboLights MTBLS733 with msdial_params.txt and msflo_params.ini in data/, invoking 'nextflow run main.nf -profile docker' and capturing logs/execution.log each time. Reset the Nextflow work directory between runs so each execution starts from identical inputs, eliminating cache reuse as a confounder. Collect the .tsv peak tables produced by MS-DIAL and MS-FLO from the results directory for every run. Compare the runs pairwise by checksum or row-level diff of the peak tables and record the agreement (e.g. in run_comparison.tsv). Acceptance criterion: every pair of runs must be identical, and totals should match the paper's reported values — MS-DIAL 25,447 total peaks with 1,031 true positives (4.05%) and MS-FLO 25,414 total peaks with 1,030 true positives (4.05%).

## Related tools

- **Nextflow4MS-DIAL** (The workflow under test — executed repeatedly on MTBLS733 data to generate the peak tables being compared) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Execution engine; 'nextflow run main.nf -profile docker' is repeated five times with work-directory resets between runs) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (Produces the MS-DIAL peak table (one of the compared outputs); containerized version 4.48 used in the reproducibility check) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Docker** (Container runtime via the docker profile, ensuring each of the five runs uses the same MS-DIAL/MS-FLO software environment) — https://docs.docker.com/engine/installation/

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- All five runs produce byte-identical (checksum-matching) or row-level-identical MS-DIAL and MS-FLO .tsv peak tables
- MS-DIAL total peak count equals 25,447 with 1,031 true positives (4.05%), matching the paper's reported values
- MS-FLO total peak count equals 25,414 with 1,030 true positives (4.05%), matching the paper's reported values
- Each run's logs/execution.log records a completed workflow with identical Nextflow and workflow versions, parameter settings, and container information
- The work directory was reset between runs so no cached processes were reused

## Limitations

- The check was performed on a small test set of only 10 samples, so identical results are demonstrated for this dataset, not in general
- The authors did not quantitatively measure scalability or reproducibility beyond this five-run identical-results demonstration
- MS-FLO must be downloaded from the Internet rather than bundled, so network changes could affect run-to-run consistency
- Workflow components such as MS-DIAL are updated frequently and updates are monitored manually; results are only reproducible with the pinned container (MS-DIAL 4.48)

## Evidence
