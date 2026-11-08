---
name: metabolomics-repository-study-retrieval
description: Use when a study ID from MetaboLights or Metabolomics Workbench is available
  and the raw/NMR acquisition files need to be fetched, harmonized into a standardized
  directory format, and made ready-to-use for data analysis platforms.
when_to_use_negative:
- The data set is already in an analysis-ready feature/peak table format, so no file-system
  harmonization is needed
- The goal is statistical analysis or biological interpretation itself — CloMet only
  fills the data accessibility gap and explicitly does not implement data processing
  algorithms
- Re-analysis for classification problems is required but class metadata is not available
  along with the metabolomics data in the public repository
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0154
tools:
- name: CloMet
  role: Pipeline (LoadDataset) that downloads repository data by study ID and converts
    raw/NMR files into the standardized output format
  repo: https://github.com/rmallol/clomet
- name: Docker
  role: Execution container for running the CloMet pipeline reproducibly
- name: MetaboLights
  role: Source repository of raw/NMR metabolomics data sets retrieved by study ID
- name: Metabolomics Workbench
  role: Source repository of raw/NMR metabolomics data sets retrieved by study ID
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:646b7c475080a03a48290572f9219ea6c9aea5eb4f207c9744c83f420fd8f5e0
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - Docker
  - MetaboLights
  - Metabolomics Workbench
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolomics-repository-study-retrieval

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Retrieve raw and NMR-based metabolomics data sets from public repositories (MetaboLights, Metabolomics Workbench) by study identifier and convert them into a harmonized, analysis-ready file format using the CloMet pipeline, so downstream platforms like MetaboAnalyst or Workflows4Metabolomics can consume them directly.

## When to use

Use when a study ID from MetaboLights or Metabolomics Workbench is available and the raw/NMR acquisition files need to be fetched, harmonized into a standardized directory format, and made ready-to-use for data analysis platforms.

## When NOT to use

- The data set is already in an analysis-ready feature/peak table format, so no file-system harmonization is needed
- The goal is statistical analysis or biological interpretation itself — CloMet only fills the data accessibility gap and explicitly does not implement data processing algorithms
- Re-analysis for classification problems is required but class metadata is not available along with the metabolomics data in the public repository

## Inputs

- MetaboLights study ID (public data set)
- Metabolomics Workbench study ID (public data set)
- Private/local metabolomics data (via the LOCAL tab)
- Docker with the CloMet image built from github.com/rmallol/clomet

## Outputs

- Analysis-ready output data set in CloMet's custom directory-based standard format, directly usable in MetaboAnalyst or Workflows4Metabolomics
- Per-study success/failure status (e.g., benchmark_results.csv with correct-output counts per cohort)

## How to apply

Run the CloMet LoadDataset pipeline inside the provided Docker container for each study ID; CloMet downloads the raw/NMR acquisition files from MetaboLights or Metabolomics Workbench and converts the original file system into a custom directory-based standard that harmonizes the different repository formats. The output must satisfy the format requirements of existing analysis tools such as NMR_Read, NMR_Preprocessing, or NMR_Bucketing in Workflows4Metabolomics, and be directly loadable in MetaboAnalyst. Users with private/local data can use the same conversion via the LOCAL tab instead of a repository study ID. Judge success per study: a run is 'correct' when it produces a correct, ready-to-use output data set; in the benchmark, 38/41 accepted data sets (92.7%) and 38/52 repository studies (73.1%) passed, so failures should be treated as plausible per-study outcomes (e.g., missing class metadata) rather than pipeline bugs.

## Related tools

- **CloMet** (Pipeline (LoadDataset) that downloads repository data by study ID and converts raw/NMR files into the standardized output format) — https://github.com/rmallol/clomet
- **Docker** (Execution container for running the CloMet pipeline reproducibly)
- **MetaboLights** (Source repository of raw/NMR metabolomics data sets retrieved by study ID)
- **Metabolomics Workbench** (Source repository of raw/NMR metabolomics data sets retrieved by study ID)

## Examples

```
docker run -v $(pwd)/output:/app/output clomet LoadDataset --study-id MTBLS1
```

## Evaluation signals

- The converted output data set loads directly in MetaboAnalyst or in Workflows4Metabolomics tools (NMR_Read, NMR_Preprocessing, NMR_Bucketing) without manual reformatting
- Per-study status check confirms a correct, ready-to-use output data set was produced for each run
- Aggregate correct-output rate across a cohort falls in the reported range: 92.7% (38/41) for accepted data sets, 73.1% (38/52) for all repository studies
- The output directory structure matches CloMet's internal standard that harmonizes MetaboLights and Metabolomics Workbench formats
- The run was executed inside the Docker container so the retrieval is reproducible

## Limitations

- CloMet focuses on NMR-based untargeted metabolomics data sets; non-NMR raw data coverage may be limited
- If information about classes is not available along with the metabolomics data in public repositories, re-analysis oriented to classification problems won't be feasible
- CloMet does not itself implement data processing algorithms — conversion only; downstream processing is out of scope
- Maintenance and extension depend on the open-source developer community; integration with existing analysis platforms was still a future plan
- Not all retrieval runs succeed — 15/52 repository studies in the benchmark failed to produce a correct output

## Evidence
