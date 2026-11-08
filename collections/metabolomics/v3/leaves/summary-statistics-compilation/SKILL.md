---
name: summary-statistics-compilation
description: Use when reproducing or tabulating feature/annotation throughput statistics
  of a reaction-guided metabolomics workflow (DMS, RCS, SSN layers) over LC-HRMS data
  from enzymatic biotransformation assays.
when_to_use_negative:
- The analysis goal is novel metabolite discovery/annotation itself rather than verification
  of reported throughput statistics
- Input data are targeted LC-MS runs or reference-library bottom-up workflows where
  the DMS/RCS/SSN differential spaces are not defined
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CMDN
  role: Triple-layered (DMS/RCS/SSN) MS data processing framework whose outputs the
    summary statistics quantify
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Instrumental platform producing the feature table and PRM MS2 spectra being
    counted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:967ab2589ea912c35a1e9270b11a8813ecd7688788c95ac3378585b2ab57689b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# summary-statistics-compilation

## Summary

Compiles the reported summary statistics table for the CMDN triple-layered annotation pipeline (DMS/RCS/SSN) applied to the 1,021-pesticide human liver S9 LC-HRMS incubation data, reproducing counts, match rates, transformation rates, and reaction-class prevalences.

## When to use

Use when reproducing or tabulating feature/annotation throughput statistics of a reaction-guided metabolomics workflow (DMS, RCS, SSN layers) over LC-HRMS data from enzymatic biotransformation assays.

## When NOT to use

- The analysis goal is novel metabolite discovery/annotation itself rather than verification of reported throughput statistics
- Input data are targeted LC-MS runs or reference-library bottom-up workflows where the DMS/RCS/SSN differential spaces are not defined

## Inputs

- Processed LC-HRMS feature table from the human liver S9 enzymatic assay of 1,021 pesticides
- CMDN reaction rules (n = 290) applied to the curated PMDB (N = 492, Table S2 sheet 1)
- 5,358 PRM MS2 spectra for realignment against RCS candidates
- R environment with CMDN dependencies (tidyverse, CluMSID, OrgMassSpecR, MSMSsim, msentropy)

## Outputs

- cmdn_summary_statistics.csv with layer-wise counts, match rates, per-pesticide transformation rates, and reaction prevalences
- Reproduction check: 793 upregulated features, 2,886 nonredundant annotations, 87.3% MS1-MS2 match rate, 77.1% unique-spectra fraction

## How to apply

Load the processed LC-HRMS feature table from the human liver S9 enzymatic assay of 1,021 pesticides and run the CMDN differential metabolite space (DMS) to quantify total features (32,827) and upregulated features (793). Apply the reaction-based pseudo-MS1 candidature space (RCS) with the 290 CMDN reaction rules against the curated PMDB (N = 492) to annotate 6,181 MS1 features, retaining 3,813 after deduplication. Build the MS2 spectrum similarity network (SSN): realign the 5,358 PRM spectra, derive 2,886 nonredundant annotations, and compute the 87.3% MS1-MS2 match rate and 77.1% unique-spectra fraction. Compute per-pesticide transformation rates (median 35.3%; 533 downregulated pesticides, 52.2%) and reaction-class prevalences — oxidation 31.7%, demethylation 12.5%, deethylation 11.2% — then write everything to cmdn_summary_statistics.csv using the R tidyverse stack used by CMDN. Validation: each computed statistic must match the reported value within rounding; discrepancies flag a failed reproduction.

## Related tools

- **CMDN** (Triple-layered (DMS/RCS/SSN) MS data processing framework whose outputs the summary statistics quantify) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Instrumental platform producing the feature table and PRM MS2 spectra being counted)

## Examples

```
library(tidyverse); write_csv(cmdn_summary_statistics, "cmdn_summary_statistics.csv")
```

## Evaluation signals

- Total DMS features = 32,827 with 793 upregulated; SSN nonredundant assignments = 2,886 of 5,358 realigned PRM spectra
- MS1-MS2 match rate = 87.3%; unique-spectra fraction = 77.1%
- Per-pesticide median transformation rate = 35.3%; 533 (52.2%) pesticides downregulated
- Reaction prevalences: oxidation 31.7%, demethylation 12.5%, deethylation 11.2%
- All reproduced values match reported values within rounding; any discrepancy flags failed reproduction

## Limitations

- Reproduction depends on MS-Dial (ver. 4.80) feature tables and CMDN-compatible R packages; missing dependencies break the pipeline
- Statistics are specific to the 1,021-pesticide human liver S9 dataset and the 290-reaction-rule / PMDB (N = 492) configuration; other datasets will not match reported values

## Evidence

- [readme] README lists R dependencies and compatible software versions: [evidence span withheld — anchor status: not_found]
