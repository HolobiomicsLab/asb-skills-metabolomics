---
name: tissue-enriched-ion-proportion-analysis
description: Use when you have a peak table matrix of pixels by ion intensities with
  spatial coordinates and need to quantify the proportion of ions with higher expression
  in a tissue region versus background, as in SMQVP's QVP3 module.
when_to_use_negative:
- Input is already a preprocessed or feature-summarized matrix without per-pixel intensities,
  so tissue vs background pixel groups cannot be compared per ion
- The goal is noise ion filtering or background pixel removal rather than quantifying
  the proportion of tissue-enriched ions
- Data lacks spatial coordinates or delineated tissue/background regions, so the rank-sum
  grouping cannot be defined
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SMQVP
  role: Web application (QVP3 ion expression proportion module) that provides lasso-based
    tissue/background region selection and computes the proportion of ions with higher
    expression in the tissue region
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Runtime environment (with Shiny) in which SMQVP is implemented and the Wilcoxon
    rank-sum analysis can be run
- name: Shiny
  role: Framework powering SMQVP's interactive web interface for region selection
    and quality visualization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:6ab1919e8c151f4697269573100a59f717e150c5a6d6b64f4d96cd32b114c342
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tissue-enriched-ion-proportion-analysis

## Summary

Classifies each ion in an AFADESI spatial metabolomics peak table as tissue-enriched via a Wilcoxon rank-sum test between tissue and background pixel groups (fold change > 1, p < 0.05), reproducing SMQVP's QVP3 ion expression proportion metric (37% of 2,654 ions in the mouse brain dataset).

## When to use

Use when you have a peak table matrix of pixels by ion intensities with spatial coordinates and need to quantify the proportion of ions with higher expression in a tissue region versus background, as in SMQVP's QVP3 module.

## When NOT to use

- Input is already a preprocessed or feature-summarized matrix without per-pixel intensities, so tissue vs background pixel groups cannot be compared per ion
- The goal is noise ion filtering or background pixel removal rather than quantifying the proportion of tissue-enriched ions
- Data lacks spatial coordinates or delineated tissue/background regions, so the rank-sum grouping cannot be defined

## Inputs

- Peak table matrix (CSV/tabular) with first two columns as X and Y pixel coordinates and subsequent columns as m/z values with intensities per pixel (53,812 pixels x 2,654 ion peaks from OMIX009541)
- User-defined tissue and background pixel group assignments (region delineation, e.g. lasso-selected regions on the median pixel intensity image)

## Outputs

- Per-ion table (tissue_enriched_ions.csv) with fold change, Wilcoxon rank-sum p-value, and tissue-enriched classification flag for each of the 2,654 ions
- Overall tissue-enriched ion proportion (fraction of detected ions with fold change > 1 and p < 0.05)

## How to apply

Load the AFADESI mouse brain peak table matrix (53,812 pixels x 2,654 ion peaks) from the OMIX009541 deposit with pandas. Partition pixels into tissue and background groups using the five tissue and background regions delineated on the median pixel intensity image (in SMQVP this is done with the lasso tool on the interaction image). For each ion, run a Wilcoxon rank-sum test between tissue and background pixel intensities (scipy.stats.ranksums) and compute the fold change of median tissue intensity over median background intensity. Flag an ion as tissue-enriched when fold change > 1 AND p-value < 0.05, then compute the fraction of the 2,654 detected ions meeting both criteria. The rationale is that background pixels carry nonspecific signal, so ions significantly enriched in tissue represent genuine molecular signal rather than background contamination.

## Related tools

- **SMQVP** (Web application (QVP3 ion expression proportion module) that provides lasso-based tissue/background region selection and computes the proportion of ions with higher expression in the tissue region) — https://github.com/mzlab-research/SMQVP
- **R** (Runtime environment (with Shiny) in which SMQVP is implemented and the Wilcoxon rank-sum analysis can be run)
- **Shiny** (Framework powering SMQVP's interactive web interface for region selection and quality visualization)

## Examples

```
from scipy.stats import ranksums; stat, p = ranksums(ion[tissue_mask], ion[background_mask])
```

## Evaluation signals

- Computed tissue-enriched ion proportion falls in the range [0.35, 0.39], reproducing the paper's reported value of 37% of detected ions
- Every ion row in the output table carries fold change, p-value, and tissue-enriched classification fields
- Ion count in the analysis equals the 2,654 detected ion peaks from the AFADESI dataset
- Tissue-enriched classification is consistent with criteria: fold change > 1 AND Wilcoxon p-value < 0.05
- Tissue/background pixel groups match the five regions delineated on the median pixel intensity image

## Limitations

- Result depends on user-defined tissue and background region selection (lasso-delineated), so reproducibility requires the same region delineation as reported in the paper
- No software version or license information is stated for SMQVP, and the repository has no changelog
- Validated on a single AFADESI mouse brain dataset (positive ion mode, 53,812 pixels x 2,654 peaks); behavior on other platforms or ion modes is not demonstrated

## Evidence

- [readme] Ion Expression Proportion: Determine the proportion of ions with higher expression in the tissue region.: "Ion Expression Proportion: Determine the proportion of ions with higher expression in the tissue region."
- [results] QVP3 analysis of the AFADESI mouse brain dataset showed that 37% of the detected ions were enriched in the tissue.: [evidence span withheld — anchor status: not_found]
- [readme] This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541: "The data includes 53,812 pixels and 2,654 ion peaks. This peak table and raw data are available in the public repository https://ngdc.cncb.ac.cn/omix/release/OMIX009541."
- [intro] through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions: "through eight quality visualization points, including background consistency evaluation, noise ion filtering, intensity distribution analysis, and identification of isotopic and adduct ions"
