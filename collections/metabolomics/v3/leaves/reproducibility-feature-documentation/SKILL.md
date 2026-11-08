---
name: reproducibility-feature-documentation
description: Use when you need to characterize how reproducible and how complex a
  pipeline is (process count, parameter count, containerization) and compare it against
  a published baseline such as Metaboigniter's 20+ processes and 200+ parameters.
when_to_use_negative:
- The comparator workflow's process/parameter counts are not published or independently
  verifiable, so the baseline figures cannot be grounded
- The workflow source code (main.nf, conf/ files) is not available, so process and
  parameter enumeration would be guesswork
- The goal is algorithm performance benchmarking (peak detection accuracy, runtime
  scaling) rather than complexity/reproducibility documentation — the authors explicitly
  did not quantitatively measure scalability
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0091
tools:
- name: Nextflow4MS-DIAL
  role: Subject workflow whose processes and parameters are enumerated and whose repeat-execution
    reproducibility is verified
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Workflow engine whose process definitions and config files are parsed to count
    processes and parameters; also executes the repeated reproducibility runs
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: MS-DIAL
  role: First of the two major containerized processes whose parameters (msdial_params.txt)
    are counted and whose outputs (peak tables) are compared across runs
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:9acf4305541e7a26376cef5bd874118af930577469bc0ddd226fa3f9f2a31bf2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - MS-DIAL
  - Nextflow
  - Nextflow4MS-DIAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# reproducibility-feature-documentation

## Summary

Documents a workflow's reproducibility posture by enumerating its process/parameter complexity against a published comparator and verifying that repeated executions with identical parameters and the same container yield identical outputs.

## When to use

Use when you need to characterize how reproducible and how complex a pipeline is (process count, parameter count, containerization) and compare it against a published baseline such as Metaboigniter's 20+ processes and 200+ parameters.

## When NOT to use

- The comparator workflow's process/parameter counts are not published or independently verifiable, so the baseline figures cannot be grounded
- The workflow source code (main.nf, conf/ files) is not available, so process and parameter enumeration would be guesswork
- The goal is algorithm performance benchmarking (peak detection accuracy, runtime scaling) rather than complexity/reproducibility documentation — the authors explicitly did not quantitatively measure scalability

## Inputs

- Nextflow process definitions (main.nf, conf/base.config, conf/HiPerGator.config)
- User-facing parameter files msdial_params.txt and msflo_params.ini (from functional_test/sample_data/)
- Paper-reported comparator figures (Metaboigniter process and parameter counts)
- Execution logs from repeated runs (logs/execution.log, execution_report.html, execution_timeline.html)

## Outputs

- CSV comparison table process_parameter_comparison.csv (one row per workflow; process-count and parameter-count columns)
- Run-to-run reproducibility assessment (identical vs. divergent outputs across N repeated executions)

## How to apply

Parse the Nextflow process definitions (main.nf and conf/base.config, conf/HiPerGator.config) to enumerate every process; Nextflow4MS-DIAL contains only two major processes (MS-DIAL and MS-FLO). Extract user-facing parameters from the config files plus the shipped MS-DIAL and MS-FLO parameter files (msdial_params.txt, msflo_params.ini in functional_test/sample_data/) and count each configurable parameter. Record the paper-reported comparator figures (Metaboigniter: >20 processes, >200 parameters) as baseline values. Assemble a CSV table (process_parameter_comparison.csv) with one row per workflow and columns for process count and parameter count. Separately, verify computational reproducibility by executing the workflow N times (the paper used five runs) under identical parameter settings and the same software container (Docker or Singularity), then diff the result outputs run-to-run — identical peak counts (e.g. 25,447 MS-DIAL peaks) and tables demonstrate consistency.

## Related tools

- **Nextflow4MS-DIAL** (Subject workflow whose processes and parameters are enumerated and whose repeat-execution reproducibility is verified) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Workflow engine whose process definitions and config files are parsed to count processes and parameters; also executes the repeated reproducibility runs) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **MS-DIAL** (First of the two major containerized processes whose parameters (msdial_params.txt) are counted and whose outputs (peak tables) are compared across runs) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial

## Examples

```
nextflow run main.nf -profile functional_test > logs/execution.log
```

## Evaluation signals

- process_parameter_comparison.csv exists in valid CSV format and reports exactly 2 major processes for Nextflow4MS-DIAL, consistent with the paper's claim
- Metaboigniter comparator row exceeds the reported thresholds of 20 processes and 200 parameters
- Five repeated executions under identical parameters and the same container produce identical outputs (demonstrated in the paper with 25,447 MS-DIAL peaks per run)
- Execution logs record the metadata needed to reproduce a run: Nextflow and workflow versions, parameter settings, resource allocation, and container information
- Result files and example logs are committed to the repository's 'results' folder so third parties can diff their own runs against them

## Limitations

- The authors did not quantitatively measure scalability or reproducibility beyond the five-run identical-output check
- Validation used only 10 samples (five per group) from MetaboLights MTBLS733 / the Li et al. dataset
- Metaboigniter process/parameter counts are paper-reported baselines, not independently re-enumerated
- The workflow has not been tested on Apple silicon (M-series) systems, so container-based reproducibility is only established for Intel-based macOS and Linux (HiPerGator, Red Hat Enterprise Linux 8.8)

## Evidence
