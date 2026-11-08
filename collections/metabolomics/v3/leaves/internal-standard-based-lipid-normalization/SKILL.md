---
name: internal-standard-based-lipid-normalization
description: Use when a lipidomics experiment (LipidSearch/LIQUID output or Metabolomics
  Workbench data) includes internal lipid standards and per-lipid absolute concentrations
  are needed before differential abundance analysis.
when_to_use_negative:
- The experiment lacks internal lipid standards — ADViSELipidomics normalizes 'when
  the experiment includes internal lipid standards', so without them only unnormalized
  preprocessing is available.
- Input is already a normalized concentration table or non-lipidomics feature table
  that does not come from LipidSearch, LIQUID, or Metabolomics Workbench output.
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: ADViSELipidomics
  role: Shiny app that performs the internal-standard-based normalization of the lipid
    data matrix, computing per-sample percentage of recovery and normalized concentrations
    per lipid and sample
  repo: https://github.com/ShinyFabio/ADViSELipidomics
- name: LipidSearch
  role: Vendor software whose identification/quantification output files are the raw
    input to the normalization step (Case Study 1)
- name: metabolomicsWorkbenchR
  role: Provides do_query() to download the Metabolomics Workbench lipidomic experiment
    (healthy vs diabetic serum, Case Study 2) that is then normalized
- name: Shiny
  role: Framework in which the ADViSELipidomics normalization workflow is implemented
    and launched
  repo: https://github.com/ShinyFabio/ADViSELipidomics
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac706
    title: 'ADViSELipidomics: a workflow for analyzing lipidomics data'
  source_content_hash: sha256:9edd5fc137915b6f865bc7abe470ce2a2f8f5625828f6fa54d7279a11fa36bd3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShinyFabio/ADViSELipidomics
  tools:
  - ADViSELipidomics
  - LipidSearch
  - Shiny
  - metabolomicsWorkbenchR
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# internal-standard-based-lipid-normalization

## Summary

Normalize a lipidomics data matrix using internal lipid standards so that each lipid in each sample is expressed as an absolute (recovery-corrected) concentration, as implemented in the ADViSELipidomics Shiny workflow.

## When to use

Use when a lipidomics experiment (LipidSearch/LIQUID output or Metabolomics Workbench data) includes internal lipid standards and per-lipid absolute concentrations are needed before differential abundance analysis.

## When NOT to use

- The experiment lacks internal lipid standards — ADViSELipidomics normalizes 'when the experiment includes internal lipid standards', so without them only unnormalized preprocessing is available.
- Input is already a normalized concentration table or non-lipidomics feature table that does not come from LipidSearch, LIQUID, or Metabolomics Workbench output.

## Inputs

- LipidSearch output files for lipid identification and quantification (e.g. Case_Study_1.zip from the ADViSELipidomics_book data_example repository)
- LIQUID output files for lipid identification and quantification
- A Metabolomics Workbench lipidomic experiment retrieved in real time via metabolomicsWorkbenchR do_query()
- Sample metadata and internal lipid standards present in the experiment

## Outputs

- Normalized concentration values per lipid and per sample (absolute concentrations)
- Percentage of recovery computed for each sample
- Normalized data matrix feeding differential abundance analysis of differentially abundant lipids

## How to apply

Load the vendor output (LipidSearch or LIQUID files, or a Metabolomics Workbench experiment downloaded via metabolomicsWorkbenchR's do_query()) into ADViSELipidomics and parse lipid species using the LIPID MAPS classification together with sample metadata. Check that the experiment contains internal lipid standards; ADViSELipidomics then computes the percentage of recovery for each sample and applies the internal-standard-based normalization to yield normalized concentration values per lipid and per sample. Use the normalized concentration matrix as the input to exploratory analysis and to differential abundance testing (e.g., the monocytes-vs-iDCs two-group comparison of Case Study 1, with batch effect correction if needed). Correctness is judged by reproducing the normalized concentration tables from the paper's case studies in the Supplementary Material.

## Related tools

- **ADViSELipidomics** (Shiny app that performs the internal-standard-based normalization of the lipid data matrix, computing per-sample percentage of recovery and normalized concentrations per lipid and sample) — https://github.com/ShinyFabio/ADViSELipidomics
- **LipidSearch** (Vendor software whose identification/quantification output files are the raw input to the normalization step (Case Study 1))
- **metabolomicsWorkbenchR** (Provides do_query() to download the Metabolomics Workbench lipidomic experiment (healthy vs diabetic serum, Case Study 2) that is then normalized)
- **Shiny** (Framework in which the ADViSELipidomics normalization workflow is implemented and launched) — https://github.com/ShinyFabio/ADViSELipidomics

## Examples

```
library("ADViSELipidomics"); run_ADViSELipidomics()
```

## Evaluation signals

- The normalized concentration table reproduces, value-for-value, the normalized concentration values per lipids and samples reported as Case Study 1 in the article's Supplementary Material.
- Each sample has a computed percentage of recovery, and the monocytes-vs-iDCs differential lipid list matches the supplementary results.
- The output matrix has one normalized concentration value per lipid-species (LIPID MAPS-parsed) and per sample, i.e. dimensions match the parsed lipid list times sample metadata rows.
- For the Metabolomics Workbench route (Case Study 2), the resulting differential lipid list and statistics qualitatively match the healthy-vs-diabetic results in the paper (no quantitative threshold reported).

## Limitations

- Normalization to absolute concentration is only possible when internal lipid standards are present in the experiment.
- The batch correction step applied downstream is not named in the paper, so the exact correction method cannot be reproduced from the text alone.
- Only 'suitable selected' Metabolomics Workbench experiments can be downloaded; an internet connection is required for that route.
- The Shiny interface appearance depends on screen size/resolution, and Chrome is the tested browser (Firefox can have problems).

## Evidence

- [abstract] When the experiment includes internal lipid standards, ADViSELipidomics can normalize the data matrix, providing normalized concentration values per lipids and samples.: "When the experiment includes internal lipid standards, ADViSELipidomics can normalize the data matrix, providing normalized concentration values per lipids and samples."
- [supplementary] ADViSELipidomics computes the percentage of recovery for each sample, providing a more accurate comparison among samples.: "ADViSELipidomics computes the percentage of recovery for each sample, providing a more accurate comparison among samples"
- [abstract] It handles the outputs from LipidSearch and LIQUID for lipid identification and quantification: "It handles the outputs from LipidSearch and LIQUID for lipid identification and quantification"
- [readme] extracts information by parsing lipid species (using LIPID MAPS classification): "extracts information by parsing lipid species (using LIPID MAPS
classification)"
- [supplementary] using the `do_query()` function from the `{metabolomicsWorkbenchR}` package: "using the `do_query()` function from the `{metabolomicsWorkbenchR}` package"
