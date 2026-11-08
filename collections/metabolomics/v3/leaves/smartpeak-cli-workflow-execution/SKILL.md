---
name: smartpeak-cli-workflow-execution
description: Use when a batch of .mzML injections plus SmartPeak session/config CSVs
  must be processed end-to-end (peak picking, calibration, QC, reporting) via the
  command line rather than the SmartPeak GUI, e.g. in scripts, CI pipelines, or Dockerized
  HPC runs.
when_to_use_negative:
- Input is already a SmartPeak feature table or a SmartPeak report CSV — use downstream
  post-processing (e.g. BFAIR) instead.
- The workflow must be interactive, with per-peak manual review in the chromatogram
  viewer; use the SmartPeak GUI, not the CLI.
- The analysis is untargeted LC-MS/MS feature-based metabolomics not requiring SmartPeak's
  targeted/semi-targeted quantitation steps — OpenMS or another tool may be more appropriate.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3315
tools:
- name: SmartPeak CLI
  role: command-line entry point that wraps the SmartPeak/OpenMS processing engine;
    equivalent to the GUI and the primary invocation surface for this skill
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: OpenMS
  role: underlying toolkit whose workflow steps SmartPeak orchestrates; SmartPeak
    supplies the missing steps (Aggregating standards, Find peaks, Align, Apply quality
    control, Write to file)
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: SmartPeak GUI
  role: graphical front-end for authoring and verifying the workflow, input files,
    and integrity checks that the CLI then executes
  repo: https://github.com/AutoFlowResearch/SmartPeak
- name: Docker
  role: containerization layer for the smartpeak-cli image, providing a reproducible
    environment on any OS or cloud/HPC
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- name: SmartPeak Server
  role: scaling companion for running the CLI workflow on high-performance-computing
    resources
  repo: https://hub.docker.com/r/autoflowresearch/smartpeak-server
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c03421
    title: SmartPeak Automates Targeted and Quantitative Metabolomics Data Processing
  source_content_hash: sha256:e9c54c568e63fe662fd2dd75510eed9e5bceca2a1b5e0a0cdce1a99296905642
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/AutoFlowResearch/SmartPeak
  tools:
  - Docker
  - OpenMS
  - SmartPeak
  - SmartPeak CLI
  - SmartPeak GUI
  - SmartPeak Server
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# smartpeak-cli-workflow-execution

## Summary

Running a SmartPeak workflow through the SmartPeak CLI (an equivalent of the SmartPeak GUI), including Docker-containerized execution and workflow step cherry-picking, so that capillary-, GC-, and LC-MS(/MS) metabolomics data are processed from raw mzML to quality-controlled quantitation.

## When to use

Use when a batch of .mzML injections plus SmartPeak session/config CSVs must be processed end-to-end (peak picking, calibration, QC, reporting) via the command line rather than the SmartPeak GUI, e.g. in scripts, CI pipelines, or Dockerized HPC runs.

## When NOT to use

- Input is already a SmartPeak feature table or a SmartPeak report CSV — use downstream post-processing (e.g. BFAIR) instead.
- The workflow must be interactive, with per-peak manual review in the chromatogram viewer; use the SmartPeak GUI, not the CLI.
- The analysis is untargeted LC-MS/MS feature-based metabolomics not requiring SmartPeak's targeted/semi-targeted quantitation steps — OpenMS or another tool may be more appropriate.

## Inputs

- .mzML raw data files (e.g. FIAMS_FullScan_Unknowns, DDA, GC-MS SIM acquisitions)
- sequence CSV file (e.g. sequence_file.png example in the SmartPeak GUI workflow)
- SmartPeak session input files: TransitionGroup/Transition/Component/ComponentGroup CSVs, Quant Methods, feature filters/QC config
- Docker image autoflowresearch/smartpeak-cli (optional but documented)

## Outputs

- Picked and filtered features with meta-values (feature table, heatmap, matrix views in the GUI equivalent)
- Calibration curve fits and quantitation results (Quant Methods / Calibrators views)
- Report CSV with user-selected variables (Actions | Report dialog)
- Stored feature sets such as STORE_FEATURE_BACKGROUND_ESTIMATIONS output
- SmartPeak logs (Windows: C:\\Users\\<user>\\AppData\\Local\\SmartPeak; Linux/Mac: ~/.SmartPeak)

## How to apply

Assemble a session folder per SmartPeak's examples (src/examples/data) containing the sequence CSV, input files, and .mzML acquisitions, then launch the autoflowresearch/smartpeak-cli Docker container with the data volume mounted (docker run --rm -ti -v C:/data:/sample-data autoflowresearch/smartpeak-cli:latest bash). Edit the workflow by cherry-picking steps from the predefined sets — e.g. LOAD_RAW_DATA, PICK_MRM_FEATURES, SELECT_FEATURES, ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES, FILTER_FEATURES_BACKGROUND_INTERFERENCES, STORE_FEATURES — and verify step/input-file integrity before executing. Rationale: the CLI mirrors GUI functionality ('SmartPeak CLI provides an equivalent of SmartPeak GUI application') and builds on OpenMS, which natively covers most workflow steps except 'Aggregating standards, Find peaks, Align, Apply quality control, Write to file' — steps that SmartPeak implements itself. Judge success by the integrity-check pass (Actions | Integrity checks in the GUI equivalent), a completed run log, and a valid feature/quantitation report. Validate against example datasets like FIAMS_FullScan_Unknowns or DDA.

## Related tools

- **SmartPeak CLI** (command-line entry point that wraps the SmartPeak/OpenMS processing engine; equivalent to the GUI and the primary invocation surface for this skill) — https://github.com/AutoFlowResearch/SmartPeak
- **OpenMS** (underlying toolkit whose workflow steps SmartPeak orchestrates; SmartPeak supplies the missing steps (Aggregating standards, Find peaks, Align, Apply quality control, Write to file)) — https://github.com/AutoFlowResearch/SmartPeak
- **SmartPeak GUI** (graphical front-end for authoring and verifying the workflow, input files, and integrity checks that the CLI then executes) — https://github.com/AutoFlowResearch/SmartPeak
- **Docker** (containerization layer for the smartpeak-cli image, providing a reproducible environment on any OS or cloud/HPC) — https://hub.docker.com/r/autoflowresearch/smartpeak-cli
- **SmartPeak Server** (scaling companion for running the CLI workflow on high-performance-computing resources) — https://hub.docker.com/r/autoflowresearch/smartpeak-server

## Examples

```
docker run --rm -ti -v /home/user/FIAMS_FullScan_Unknowns:/sample-data autoflowresearch/smartpeak-cli:latest smartpeakcli --session /sample-data
```

## Evaluation signals

- Integrity-check pass ('Actions | Integrity checks' equivalent) before workflow execution
- Log file written under the default location (C:\\Users\\<user>\\AppData\\Local\\SmartPeak or ~/.SmartPeak) with a completed-run status and per-step progress
- Output report CSV opens cleanly and contains the user-selected report variables for each injection/transition
- For targeted LC-MS/MS (e.g. 5500 QTRAP RapidRIP) datasets, feature counts per injection fall in the expected range reported for the equivalent example dataset (e.g. ~574 features Spike-in, ~5,050 serum)
- STORE_* steps (e.g. STORE_FEATURES, STORE_FEATURE_BACKGROUND_ESTIMATIONS) produce persisted tables that can be re-loaded via pyOpenMS/mzTab for BFAIR post-processing

## Limitations

- Only applies to CE-, GC-, and LC-MS(/MS), and HPLC data for targeted/semi-targeted metabolomics, lipidomics, and fluxomics experiments — not general untargeted LC-MS/MS DDA/DIA feature detection.
- GUI result views are capped (features table limited to 5000 entries; chromatograms to 9000 points) for performance, so large cohorts must be consumed via the CLI report output rather than interactive viewing.
- Docker volumes must be mapped correctly (e.g. -v C:/data:/sample-data) or the CLI will not find the sequence folder.
- SmartPeak Server must be deployed for HPC-scale runs; the plain CLI is not documented as distributing work across nodes.
- The workflow automates 'all steps from peak detection and integration over calibration curve optimization, to quality control reporting' but does not itself perform untargeted feature-based FIA-MS post-processing (that is BFAIR's role).

## Evidence

- [results] Current versions of OpenMS support most steps except: Aggregating standards, Find peaks, Align, Apply quality control, Write to file. These are now implemented in SmartPeak: [evidence span withheld — anchor status: not_found]
- [methods] * - STORE_FEATURES
  * - ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES: "* - STORE_FEATURES
  * - ESTIMATE_FEATURE_BACKGROUND_INTERFERENCES"
- [readme] Results will be exported to: [evidence span withheld — anchor status: not_found]
