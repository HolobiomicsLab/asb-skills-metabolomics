---
name: r-package-workflow-execution
description: Use when an agent must run an R-based metabolomics pre-processing pipeline
  on an MS-DIAL feature abundance table plus sample annotation file, and reproduce
  documented filter counts (e.g. 548/604 kept after m/z filtering, 303/539 after CV-ratio
  filtering).
when_to_use_negative:
- Input is already a normalized, filtered feature matrix — the margheRita pre-processing
  filters assume raw MS-DIAL output with m/z, retention time and QC-sample annotations
- Data are not LC-MS/MS untargeted metabolomics features (e.g. proteomics or targeted
  assay tables) — the package targets LC-MS/MS profiling including DIA data
- The goal is statistical testing, pathway analysis (ORA/MSEA), or metabolite identification
  rather than pre-processing/quality filtering
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: margheRita
  role: Primary R package providing the pre-processing functions executed in this
    workflow (m_z_filtering, filter_NA, CV-ratio filtering, probabilistic quotient
    normalization)
  repo: https://github.com/emosca-cnr/margheRita
- name: MS-Dial
  role: Upstream tool that generates the feature abundance table consumed by margheRita
- name: notame
  role: Alternative R workflow for non-targeted LC-MS preprocessed data (drift correction,
    quality metrics, imputation); referenced for comparison, not required to execute
    this skill
  repo: https://github.com/antonvsdata/notame
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1101/2024.06.20.599545
    title: 'MargheRita: streamlining MS-DIAL output analysis and metabolite identification
      in R'
  source_content_hash: sha256:f6e8e874cfec3be4062954dda440d2b9a0a9a18f96836bb027e605e1fb32536e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/emosca-cnr/margheRita
  tools:
  - MS-Dial
  - R
  - margheRita
  - notame
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# r-package-workflow-execution

## Summary

Execute the margheRita R package's LC-MS/MS untargeted metabolomics workflow on MS-DIAL-derived feature tables, reproducing documented pre-processing filters (m/z filtering, NA filtering, CV-ratio filtering samples vs QCs) and normalization steps with verifiable feature counts.

## When to use

Use when an agent must run an R-based metabolomics pre-processing pipeline on an MS-DIAL feature abundance table plus sample annotation file, and reproduce documented filter counts (e.g. 548/604 kept after m/z filtering, 303/539 after CV-ratio filtering).

## When NOT to use

- Input is already a normalized, filtered feature matrix — the margheRita pre-processing filters assume raw MS-DIAL output with m/z, retention time and QC-sample annotations
- Data are not LC-MS/MS untargeted metabolomics features (e.g. proteomics or targeted assay tables) — the package targets LC-MS/MS profiling including DIA data
- The goal is statistical testing, pathway analysis (ORA/MSEA), or metabolite identification rather than pre-processing/quality filtering

## Inputs

- Feature abundance table (tab-delimited, MS-DIAL-derived: feature identifiers, m/z, retention times, MS/MS spectra, abundances)
- Sample annotation file (tab-delimited, columns: id, injection_order, batch, class, biological_rep, technical_rep) with class labels distinguishing QC samples from biological samples
- mRList object derived from the Urine dataset (Zenodo doi 10.5281/zenodo.11243781)

## Outputs

- Filtered feature table (e.g. cv_filtered_features.tsv) with only metabolites passing the applied filters
- Count summary of kept vs removed features per filter step (e.g. cv_filter_summary.txt: 303 kept / 236 removed by CV-ratio filter; 548 kept / 56 removed by m/z filter)
- Updated mRList object reflecting the filtered feature set

## How to apply

Load the margheRita package in R and read the two required tab-delimited inputs: a feature data file (feature identifiers, m/z, retention times, MS/MS spectra, abundances) and a sample annotation file with columns id, injection_order, batch, class, biological_rep, technical_rep; the demo data derive from the 'Urine' dataset (Zenodo 10.5281/zenodo.11243781) held in an mRList object. Run pre-processing functions in the documented order: sample filtering (samples with >= 100 metabolites), feature filtering (features occurring in >= 3 samples), m_z_filtering() with default lower_quality_mass_acc = 4 and upper_quality_mass_acc = 8 (removes features whose m/z decimal value lies within [4, 8]), filter_NA with min_metab_in_sample / min_sample_with_metab thresholds, CV-ratio filtering keeping features whose CV(no-QC samples)/CV(QC samples) exceeds a threshold (default 1), and probabilistic quotient normalization. Optional imputation replaces NA values with a random number between 10%-25% of the minimum value of the feature. After each filter, count retained vs removed features and compare against the documentation's reported counts (e.g. 548 with appropriate m/z values and 56 removed; 303 of 539 metabolites kept by the CV filter) as validation before writing the filtered feature table and a summary of kept/removed counts to output artifacts.

## Related tools

- **margheRita** (Primary R package providing the pre-processing functions executed in this workflow (m_z_filtering, filter_NA, CV-ratio filtering, probabilistic quotient normalization)) — https://github.com/emosca-cnr/margheRita
- **MS-Dial** (Upstream tool that generates the feature abundance table consumed by margheRita)
- **notame** (Alternative R workflow for non-targeted LC-MS preprocessed data (drift correction, quality metrics, imputation); referenced for comparison, not required to execute this skill) — https://github.com/antonvsdata/notame

## Examples

```
Rscript -e 'library(margheRita); mrl <- read_mRList("urine_demo"); mrl <- m_z_filtering(mrl, lower_quality_mass_acc = 4, upper_quality_mass_acc = 8); mrl <- cv_filtering(mrl, threshold = 1)'
```

## Evaluation signals

- m/z filter on the demo mRList retains exactly 548 features with appropriate m/z values and removes 56 features without appropriate m/z values
- CV-ratio filter with default threshold 1 on normalized Urine profiles retains exactly 303 of 539 metabolites
- Sample/feature NA filter counts match documentation: samples with >= 100 metabolites 243/243; features occurring in >= 3 samples 604/604
- Filtered output table contains no features whose m/z decimal value falls within the default [4, 8] interval
- Every retained metabolite has CV(no-QC samples)/CV(QC samples) strictly higher than the default threshold of 1

## Limitations

- Reported counts (303/539, 548/56, 604/604, 243/243) are specific to the demonstration Urine dataset; other datasets will yield different retained-feature counts
- Imputation is skipped when no NAs remain after filtering ('No NAs: imputation not performed.'), so imputed-value outputs are not guaranteed
- Default thresholds (m/z decimal interval [4, 8], CV-ratio threshold 1, min 100 metabolites/sample, min 3 samples/feature) may be inappropriate for datasets with different mass accuracy or QC design
- The notame repository is migrated and no longer maintained; issues/pull requests there are not monitored

## Evidence

- [intro] m/z filter defaults: "`m_z_filtering()` remove features with m/z that have decimal value within [4, 8] (by default)"
- [intro] CV ratio filter: "only features that have a CV ratio between no-QC samples and QC sample higher than a given threshold (by default 1) are kept"
- [intro] input files: "It requires two text files in tab-delimited format"
- [readme] package scope: "a series of pre-processing functions (quality control, filtering and normalization) with a particular focus on methods specifically recommended for metabolomic profiles"
- [intro] imputation rule: "the imputation is performed replacing NA values with a random number, calculated between 10%-25% of the minimum value of the feature."
