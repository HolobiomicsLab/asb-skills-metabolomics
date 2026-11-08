---
name: cross-platform-peak-table-comparison
description: Decision support for validating that a containerized LC-MS metabolomics
  workflow produces identical peak tables when run on different platforms (e.g. macOS
  with Docker vs Linux HPC with Singularity).
when_to_use_negative:
- The data are already a feature/peak table — use a direct comparison of existing
  tables instead of re-running the full MS-DIAL -> MS-FLO workflow.
- The platform or software container differs (e.g. testing on Apple silicon M-series,
  which the workflow has not been validated for).
- The goal is a new metabolite identification rather than verifying reproducibility
  of an existing parameter set — the optional metabolite identification step and parameter
  tuning dominate and the identical-result expectation no longer holds.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Nextflow4MS-DIAL
  role: The workflow under test; orchestrates MS-DIAL and MS-FLO inside containers
    on local and HPC platforms.
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- name: Nextflow
  role: Executes the workflow with `-profile docker` or `-profile singularity` to
    target local or HPC execution.
  repo: https://www.nextflow.io/
- name: MS-DIAL
  role: Performs LC-focused peak detection, m/z and retention time identification,
    and intensity calculation; version 4.48 inside the container.
- name: MS-FLO
  role: Applies duplicate peak flagging, adduct and isotope flagging, peak alignment,
    and contaminant ion removal to the MS-DIAL peak table.
- name: Docker
  role: Container runtime used for the local desktop (macOS) execution profile.
  repo: https://docs.docker.com/engine/installation/
- name: Singularity
  role: Container runtime used for the Linux HPC (HiPerGator/SLURM) execution profile.
  repo: https://www.sylabs.io/guides/3.0/user-guide/
- name: MetaboLights MTBLS733
  role: Public reference dataset (10-sample Li et al. 2018 LC-HRMS set) providing
    the input mzML files for the reproducibility test.
  repo: https://www.ebi.ac.uk/metabolights/editor/MTBLS733/protocols
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:bb9629c07b68a09ccb87aeb3bc61639f216007c51efffc96afc1807b97030be8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - Docker
  - MS-DIAL
  - MS-FLO
  - MetaboLights MTBLS733
  - Nextflow
  - Nextflow4MS-DIAL
  - Singularity
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# cross-platform-peak-table-comparison

## Summary

Reproduces the peak tables produced by a containerized Nextflow4MS-DIAL run (MS-DIAL -> MS-FLO) on a local desktop (Docker profile) and a Linux HPC (Singularity profile) and verifies that the outputs are identical, demonstrating computational reproducibility across platforms.

## When to use

Decision support for validating that a containerized LC-MS metabolomics workflow produces identical peak tables when run on different platforms (e.g. macOS with Docker vs Linux HPC with Singularity).

## When NOT to use

- The data are already a feature/peak table — use a direct comparison of existing tables instead of re-running the full MS-DIAL -> MS-FLO workflow.
- The platform or software container differs (e.g. testing on Apple silicon M-series, which the workflow has not been validated for).
- The goal is a new metabolite identification rather than verifying reproducibility of an existing parameter set — the optional metabolite identification step and parameter tuning dominate and the identical-result expectation no longer holds.

## Inputs

- 10-sample LC-HRMS dataset in .mzML (or .abf) format from MetaboLights MTBLS733 (Li et al. 2018), placed in data/raw_data/
- msdial_params.txt configuration file (MS-DIAL 4.48)
- msflo_params.ini configuration file (MS-FLO)
- ms1_lib.txt and ms2_lib.msp metabolite library files
- Container image shipping MS-DIAL 4.48 and MS-FLO

## Outputs

- MS-DIAL peak table (.msdial renamed to .tsv)
- MS-FLO peak table (.tsv)
- execution_report.html summarizing runtime and resource usage
- execution_timeline.html showing per-process timing
- logs/execution.log with Nextflow version, parameters, container info, and process-level logs

## How to apply

Download the reference dataset (10-sample Li et al. 2018 mzML set from MetaboLights MTBLS733) into data/raw_data/ and place the msdial_params.txt and msflo_params.ini configuration files, plus ms1_lib.txt and ms2_lib.msp libraries, in data/. Execute `nextflow run main.nf -profile docker` locally on macOS 13.5.1 (2.6 GHz 6-Core Intel Core i7, 16 GB memory) and `nextflow run main.nf -profile singularity` on HiPerGator (Red Hat Enterprise Linux 8.8) with 20 CPUs and 200 GB of memory allocated via SLURM, using identical parameters and the same container (MS-DIAL 4.48 and MS-FLO). Collect the .msdial peak tables renamed to .tsv, plus execution_report.html and execution_timeline.html from both platforms. The comparison validates that peak counts and true-positive peaks match the reference values (MS-DIAL 25,447 peaks / 1,031 true positives; MS-FLO 25,414 peaks / 1,030 true positives) and that five repeated executions produce identical results. The rationale is that software containerization via Docker and Singularity removes environment-dependent variation, so any difference in peak tables signals a container or configuration defect rather than a legitimate platform effect.

## Related tools

- **Nextflow4MS-DIAL** (The workflow under test; orchestrates MS-DIAL and MS-FLO inside containers on local and HPC platforms.) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
- **Nextflow** (Executes the workflow with `-profile docker` or `-profile singularity` to target local or HPC execution.) — https://www.nextflow.io/
- **MS-DIAL** (Performs LC-focused peak detection, m/z and retention time identification, and intensity calculation; version 4.48 inside the container.)
- **MS-FLO** (Applies duplicate peak flagging, adduct and isotope flagging, peak alignment, and contaminant ion removal to the MS-DIAL peak table.)
- **Docker** (Container runtime used for the local desktop (macOS) execution profile.) — https://docs.docker.com/engine/installation/
- **Singularity** (Container runtime used for the Linux HPC (HiPerGator/SLURM) execution profile.) — https://www.sylabs.io/guides/3.0/user-guide/
- **MetaboLights MTBLS733** (Public reference dataset (10-sample Li et al. 2018 LC-HRMS set) providing the input mzML files for the reproducibility test.) — https://www.ebi.ac.uk/metabolights/editor/MTBLS733/protocols

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- Five repeated executions on the same platform produce identical peak tables and execution reports.
- The macOS Docker run and the HiPerGator Singularity run both yield MS-DIAL peak tables with 25,447 peaks and 1,031 true positives.
- Both platforms yield MS-FLO peak tables with 25,414 peaks and 1,030 true positives.
- The generated execution_report.html and execution_timeline.html on each platform report equivalent resource usage and timeline for the same workload.
- Result files match the reference outputs deposited in the GitHub repository's `results` folder.

## Limitations

- Only a 10-sample test set was used, so reproducibility on larger or more heterogeneous cohorts is unverified.
- No quantitative measurement of scalability or reproducibility was reported; the validation is a pass/fail comparison of peak counts.
- MS-FLO must be downloaded from the Internet rather than bundled in the workflow container.
- Workflow components such as MS-DIAL are updated frequently and monitored manually, so a future MS-DIAL update could change results.
- The workflow has not been tested on Apple silicon (M-series) macOS, and there is no graphical user interface for users with limited computational experience.

## Evidence
