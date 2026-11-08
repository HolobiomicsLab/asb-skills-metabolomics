---
name: symbolic-profile-extraction
description: Use when raw ICP-MS ionome measurements (e.g. yeast knockout or HeLa
  ion concentrations in ppm) must be cleaned and encoded as line-level z-score and
  symbolic profiles before clustering, network analysis or enrichment steps.
when_to_use_negative:
- Input is already normalised and standardised data destined directly for genetic
  network extraction or element-element correlation analysis (IonFlow supports passing
  such data straight to advanced analysis)
- The goal is a downstream analysis section (exploratory PCA/hierarchical clustering,
  GO Slim annotation/enrichment, or relevance network extraction) rather than producing
  the cleaned dataset — PreProcessing must be run first and its output reused
- The dataset contains no batch structure or control lines to inform median batch
  correction, making the batch-correction parameters (method_norm/control_use/control_lines)
  inapplicable as configured
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: IonFlow
  role: R package whose PreProcessing function performs the log-transformation, median
    batch correction, outlier detection (Q1-3*IQ / Q3+3*IQ outer fences), standardisation
    and thres_symb symbolisation
  repo: github.com/wanchanglin/ionflow
- name: Galaxy
  role: Web platform hosting IonFlow so pre-processing is accessible via a simple
    user-friendly interface; installed via the Galaxy Tool Shed entry
  repo: https://usegalaxy.org/
- name: R
  role: Language in which the IonFlow PreProcessing pipeline is implemented and invoked
    (library(IonFlow) after devtools::install_github)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-021-01841-z
    title: 'IonFlow: a galaxy tool for the analysis of ionomics data sets'
  source_content_hash: sha256:21a9ce016961eefc5b64edd61fe72ddc333940478200d5dd44bab372af5b9326
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - IonFlow
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# symbolic-profile-extraction

## Summary

Converts raw ICP-MS ionomics concentration data (e.g. ppm values for Ca44, Cd111, Fe56, K39, etc.) into per-line z-score and symbolic profiles via IonFlow's PreProcessing function, which chains log-transformation, batch correction, outlier removal, standardisation and threshold-based symbolisation.

## When to use

Use when raw ICP-MS ionome measurements (e.g. yeast knockout or HeLa ion concentrations in ppm) must be cleaned and encoded as line-level z-score and symbolic profiles before clustering, network analysis or enrichment steps.

## When NOT to use

- Input is already normalised and standardised data destined directly for genetic network extraction or element-element correlation analysis (IonFlow supports passing such data straight to advanced analysis)
- The goal is a downstream analysis section (exploratory PCA/hierarchical clustering, GO Slim annotation/enrichment, or relevance network extraction) rather than producing the cleaned dataset — PreProcessing must be run first and its output reused
- The dataset contains no batch structure or control lines to inform median batch correction, making the batch-correction parameters (method_norm/control_use/control_lines) inapplicable as configured

## Inputs

- Raw ionomics data frame of ion concentrations in ppm, OD-adjusted (e.g. IonData from the S. cerevisiae haploid knockout benchmark: 1454 mutants, 14 ions, 2/4/8 replicas per mutant)
- Optional vector of pre-defined ion standard deviations computed from control genes/strains
- Batch structure with a control strain measured in every batch (e.g. YDL227C)

## Outputs

- data.line.zscores table (line-level standardised z-score profiles)
- data.line.symb table (symbolic profiles at threshold thres_symb)
- Diagnostic stats tables: stats.raw_data, stats.outliers, stats.median_batch_corrected_data
- Diagnostic plots supporting outlier and batch-correction inspection

## How to apply

Load the raw concentration data frame (e.g. IonData: 14 ions such as Ca, Cd, Co, Cu, Fe, K, Mg, Mn, Mo, Na, Ni, P, S, Zn measured across 1454 single-gene haploid yeast knockouts, optionally with a pre-defined standard deviation vector computed from control strains such as the YDL227C control measured in every batch). Run the PreProcessing function, which applies log-transformation of concentration values, then median batch correction where data are scaled to the per-ion, per-batch median (configured via method_norm/control_use/control_lines). Remove extreme outliers using outer fences defined as Q1 - 3*IQ and Q3 + 3*IQ, or mad/IQR-based detection with thres_outl. Standardise features using std, mad or custom scaling, then extract symbolic profiles by setting the symbolisation threshold thres_symb, producing data.line.zscores and data.line.symb plus stats tables and diagnostic plots. The cleaned dataset feeds the downstream exploratory, clustering (GO Slim/GO enrichment) and network-analysis sections, which can be run in any order after pre-processing.

## Related tools

- **IonFlow** (R package whose PreProcessing function performs the log-transformation, median batch correction, outlier detection (Q1-3*IQ / Q3+3*IQ outer fences), standardisation and thres_symb symbolisation) — github.com/wanchanglin/ionflow
- **Galaxy** (Web platform hosting IonFlow so pre-processing is accessible via a simple user-friendly interface; installed via the Galaxy Tool Shed entry) — https://usegalaxy.org/
- **R** (Language in which the IonFlow PreProcessing pipeline is implemented and invoked (library(IonFlow) after devtools::install_github))

## Examples

```
devtools::install_github("AlinaPeluso/MetaboFlow", subdir="IonFlow"); library(IonFlow); pre_proc <- PreProcessing(data=IonData, stdev=pre_defined_sd)
```

## Evaluation signals

- stats.median_batch_corrected_data shows median 0 for every ion after log-transform plus per-ion/per-batch median scaling (e.g. Ca median 0, Mg median 0 in the README table)
- stats.outliers per-ion counts satisfy not_outlier + outlier = total measurements, with outlier percentages in the 0.01–0.22 range seen for the yeast benchmark (e.g. Ca 305/9999 = 0.22, Mg 8/9999 = 0.01)
- stats.raw_data reports ion min/quartiles/mean/variance with no missing values, matching the README statement that the yeast ICP-MS data contain no missing values
- data.line.symb values are consistent with data.line.zscores under the chosen thres_symb threshold (symbolisation bins correspond to the z-score scale)
- The cleaned output dataset loads successfully into the downstream exploratory, clustering and network sections of the IonFlow package

## Limitations

- The README documentation is truncated, so the exact default value of the symbolisation threshold thres_symb and downstream standardisation details are not documented in the provided context
- The article reports no quantitative benchmark results or performance metrics for the two case studies (S. cerevisiae and HeLa ionomes), so expected output quality cannot be numerically validated from the paper
- No version number is stated for IonFlow or Galaxy, and no explicit limitations/caveats are discussed in the paper's discussion section
- Outlier removal discards samples judged unreliable (up to ~22% for Ca in the benchmark), which could bias downstream line-level profiles if applied uncritically

## Evidence

- [readme] README pre-processing rationale: [evidence span withheld — anchor status: external_doc]
- [readme] README pre-processing ordering and inputs: [evidence span withheld — anchor status: external_doc]
- [other] Source card: symbolic profile extraction step: [evidence span withheld — anchor status: not_found]
- [intro] Article intro: tool purpose: "we developed IonFlow, an R-based Galaxy tool designed for the analysis of ionome data sets from ICP-MS experiments"
- [discussion] Article discussion: flexibility for pre-processed input: "IonFlow is very versatile and it can be used to process raw data as well as directly process normalised and standardised data"
