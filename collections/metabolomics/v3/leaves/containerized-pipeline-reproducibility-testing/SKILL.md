---
name: containerized-pipeline-reproducibility-testing
description: Use when validating that a containerized Nextflow LC-MS metabolomics
  pipeline (e.g. .mzML inputs through MS-DIAL and MS-FLO) yields identical run-to-run
  outputs, or when reproducing a paper's cross-platform consistency claim.
when_to_use_negative:
- Inputs or parameter files differ between runs — the comparison requires byte-identical
  inputs and a reset work directory, otherwise diffing is meaningless
- The pipeline contains nondeterministic components or updated container versions
  (e.g. a different MS-DIAL build than 4.48), so identical outputs are not expected
- You need biological/statistical validation of peak-calling quality rather than deterministic
  run-to-run output identity
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Nextflow4MS-DIAL
  role: The workflow under test; supplies the five-run reproducibility protocol and
    reference results in its GitHub 'results' folder
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Workflow engine enabling repeated executions with Docker/Singularity profiles
    and process-level logs
  repo: https://www.nextflow.io/
- name: Docker
  role: Container runtime used for the local repeated-execution runs (-profile docker)
- name: Singularity
  role: Container runtime used for the HPC cross-platform runs (-profile singularity)
- name: MS-DIAL
  role: Peak detection whose output (25,447 peaks; 1,031 true positives) serves as
    the reference reproducibility signal
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:bc6571f60e039200b9d74d3da0ed2f6f9519636fb249a0a130e86de7c352b5eb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - Docker
  - MS-DIAL
  - MS-FLO
  - Nextflow
  - Nextflow4MS-DIAL
  - Singularity
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# containerized-pipeline-reproducibility-testing

## Summary

A verification procedure for confirming that a containerized Nextflow LC-MS metabolomics workflow (MS-DIAL 4.48 + MS-FLO) produces identical outputs across repeated executions and across platforms (local Docker vs. Singularity on HPC).

## When to use

Use when validating that a containerized Nextflow LC-MS metabolomics pipeline (e.g. .mzML inputs through MS-DIAL and MS-FLO) yields identical run-to-run outputs, or when reproducing a paper's cross-platform consistency claim.

## When NOT to use

- Inputs or parameter files differ between runs — the comparison requires byte-identical inputs and a reset work directory, otherwise diffing is meaningless
- The pipeline contains nondeterministic components or updated container versions (e.g. a different MS-DIAL build than 4.48), so identical outputs are not expected
- You need biological/statistical validation of peak-calling quality rather than deterministic run-to-run output identity

## Inputs

- 10 raw LC-MS .mzML sample files (MetaboLights MTBLS733 test dataset)
- MS-DIAL parameter file (msdial_params.txt) and MS-FLO parameter file (msflo_params.ini)
- Optional MS1/MS2 metabolite libraries (ms1_lib.txt, ms2_lib.msp)
- Docker or Singularity container shipping MS-DIAL 4.48 and MS-FLO

## Outputs

- Five sets of MS-DIAL and MS-FLO peak tables (.tsv) from repeated executions
- Pairwise checksum/row-level diff comparison of the five runs (run_comparison.tsv)
- Nextflow execution_report.html and execution_timeline.html per run
- Captured execution log (logs/execution.log)

## How to apply

Run the pipeline once with 'nextflow run main.nf -profile docker' on a fixed test dataset (10 raw LC-MS mzML samples from MetaboLights MTBLS733, Li et al. 2018), capturing the log to logs/execution.log, then repeat the identical command four more times (five runs total), resetting the Nextflow work directory between runs so each run starts from identical inputs. Collect the MS-DIAL and MS-FLO peak tables (.msdial renamed to .tsv) from the results directory of each run and compare all five runs pairwise by checksum/row-level diff, recording agreement in run_comparison.tsv. For the cross-platform variant, repeat on an HPC with 'nextflow run main.nf -profile singularity' under SLURM (e.g. 20 CPUs, 200 GB on HiPerGator, RHEL 8.8) using the identical container and parameter files (msdial_params.txt, msflo_params.ini), then diff peak tables across platforms. The run is accepted only if all executions are identical and totals match the paper's reference values: MS-DIAL 25,447 total peaks with 1,031 true positives (4.05%) and MS-FLO 25,414 total peaks with 1,030 true positives (4.05%).

## Related tools

- **Nextflow4MS-DIAL** (The workflow under test; supplies the five-run reproducibility protocol and reference results in its GitHub 'results' folder) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Workflow engine enabling repeated executions with Docker/Singularity profiles and process-level logs) — https://www.nextflow.io/
- **Docker** (Container runtime used for the local repeated-execution runs (-profile docker))
- **Singularity** (Container runtime used for the HPC cross-platform runs (-profile singularity))
- **MS-DIAL** (Peak detection whose output (25,447 peaks; 1,031 true positives) serves as the reference reproducibility signal)

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- All five runs produce byte-identical peak tables (identical checksums / empty row-level diffs) recorded in run_comparison.tsv
- MS-DIAL totals match 25,447 peaks with 1,031 true positives (4.05%); MS-FLO totals match 25,414 peaks with 1,030 true positives (4.05%)
- Per-platform peak tables and execution reports match the reference results deposited in the GitHub 'results' folder
- Nextflow and workflow versions, parameter settings, and container information appear in logs/execution.log, and the work directory was reset between runs
- Cross-platform (macOS Docker vs. HiPerGator Singularity) tables match the same reference peak counts

## Limitations

- Reproducibility was tested only on Intel-based macOS and RHEL 8.8 HPC; the workflow has not been tested on Apple silicon (M-series) systems
- The authors did not quantitatively measure scalability or reproducibility beyond the five-run test on a single 10-sample dataset
- MS-FLO must be downloaded from the Internet rather than bundled, and MS-DIAL software updates are monitored manually, so container drift can break exact reproduction

## Evidence
