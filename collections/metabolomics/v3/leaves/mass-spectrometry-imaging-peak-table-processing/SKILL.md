---
name: mass-spectrometry-imaging-peak-table-processing
description: Use when a spatial metabolomics peak table matrix (pixels x m/z ion peaks
  with X/Y coordinates) contains background pixels and randomly distributed noise
  ions that must be detected and filtered before spatial pattern recognition or clustering.
when_to_use_negative:
- Input is raw mass-spectrometry imaging data (e.g., imzML/mzML) rather than a peak
  table matrix — SMQVP requires an already peak-picked table with X/Y coordinate columns
- The analysis goal is ion identification, isotopic annotation, or metabolite database
  matching rather than quality visualization and noise/background filtering
- The dataset has no spatial coordinates or no distinguishable background region to
  delineate with the lasso tool, so background consistency and noise-score filtering
  cannot be applied
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SMQVP
  role: Web/Shiny application that performs the eight quality visualization points
    (spectra examination, background consistency, ion expression proportion, noise
    ion detection, intensity distribution, missing value distribution, isotopic peak
    ratio, adduct ion ratio) and guided preprocessing of the peak table
  repo: https://github.com/mzlab-research/SMQVP
- name: R
  role: Environment used to load the OMIX009541 peak table and run the Wilcoxon rank-sum
    tests (QVP3) and spatstat quadrat-test noise scoring (QVP4)
  repo: https://github.com/mzlab-research/SMQVP
- name: spatstat
  role: R package providing the quadrat test for Complete Spatial Randomness used
    to compute each ion's noise score as -log10(p-value)
- name: Shiny
  role: R framework in which the SMQVP graphical interface is implemented
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.3390/metabo15060354
    title: 'SMQVP: A Web Application for Spatial Metabolomics Quality Visualization
      and Processing'
  source_content_hash: sha256:a32019756316e276434df50ea870f62ef710d9233f9c1f1b5fd6e053444e49f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzlab-research/SMQVP
  tools:
  - R
  - SMQVP
  - Shiny
  - spatstat
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mass-spectrometry-imaging-peak-table-processing

## Summary

Quality assessment and preprocessing of spatial metabolomics peak table matrices using SMQVP's eight quality visualization points, including background consistency evaluation, noise ion filtering via quadrat-test noise scores, and tissue-enriched ion classification via Wilcoxon rank-sum tests.

## When to use

Use when a spatial metabolomics peak table matrix (pixels x m/z ion peaks with X/Y coordinates) contains background pixels and randomly distributed noise ions that must be detected and filtered before spatial pattern recognition or clustering.

## When NOT to use

- Input is raw mass-spectrometry imaging data (e.g., imzML/mzML) rather than a peak table matrix — SMQVP requires an already peak-picked table with X/Y coordinate columns
- The analysis goal is ion identification, isotopic annotation, or metabolite database matching rather than quality visualization and noise/background filtering
- The dataset has no spatial coordinates or no distinguishable background region to delineate with the lasso tool, so background consistency and noise-score filtering cannot be applied

## Inputs

- Peak table matrix with X/Y pixel coordinates in the first two columns followed by per-pixel ion intensities for 2,654 m/z values (53,812 pixels; AFADESI mouse brain, positive ion mode, deposit OMIX009541)
- User-delineated tissue and background region definitions selected on the median pixel intensity image

## Outputs

- Per-ion table of Wilcoxon rank-sum p-values, tissue/background fold changes, and tissue-enriched flags plus the overall enriched proportion (tissue_enriched_ions.csv)
- Per-ion m/z list with quadrat-test noise scores (noise_scores.csv)
- Noise-filtered peak intensity table retaining ions with noise score >= 90 (noise_filtered_peak_table.csv)

## How to apply

Load the peak table matrix (e.g., the AFADESI mouse brain table of 53,812 pixels x 2,654 ion peaks from OMIX009541, with X/Y coordinates in the first two columns) in R. Use SMQVP's lasso tool to delineate tissue and background regions on the interaction image built from the median pixel intensity. For tissue-enriched ion classification (QVP3), run scipy.stats.ranksums (Wilcoxon rank-sum) per ion between tissue and background pixel intensities, compute the fold change of median tissue over median background intensity, and classify ions as enriched when fold change > 1 and p < 0.05. For noise ion filtering (QVP4), run the spatstat quadrat test for Complete Spatial Randomness on each ion's spatial intensity pattern, compute the noise score as -log10(quadrat test p-value), and remove ions with noise scores below the threshold of 90. Report the retained-ion proportion (the paper reports 37% of 2,654 ions retained, and 37% of ions tissue-enriched) and save the filtered peak table for downstream clustering.

## Related tools

- **SMQVP** (Web/Shiny application that performs the eight quality visualization points (spectra examination, background consistency, ion expression proportion, noise ion detection, intensity distribution, missing value distribution, isotopic peak ratio, adduct ion ratio) and guided preprocessing of the peak table) — https://github.com/mzlab-research/SMQVP
- **R** (Environment used to load the OMIX009541 peak table and run the Wilcoxon rank-sum tests (QVP3) and spatstat quadrat-test noise scoring (QVP4)) — https://github.com/mzlab-research/SMQVP
- **spatstat** (R package providing the quadrat test for Complete Spatial Randomness used to compute each ion's noise score as -log10(p-value))
- **Shiny** (R framework in which the SMQVP graphical interface is implemented)

## Examples

```
Rscript -e 'pt <- read.csv("peak_table.csv"); p <- sapply(pt[,-(1:2)], function(x) spatstat::quadrat.test(spatstat::as.im(matrix(x,nrow=53812)))$p.value); keep <- -log10(p) >= 90; write.csv(data.frame(mz=names(p), noise_score=-log10(p)), "noise_scores.csv")'
```

## Evaluation signals

- Retained-ion proportion after the noise-score < 90 filter should fall in 35–39% of the original 2,654 ions (paper reports 37%), and every removed ion must have a noise score below 90
- Tissue-enriched ion proportion from the Wilcoxon rank-sum analysis (fold change > 1, p < 0.05) should fall in 0.35–0.39 of detected ions (paper reports 37%)
- Output CSVs must contain per-ion rows with fold change, p-value, and classification fields (tissue_enriched_ions.csv) and m/z plus noise score (noise_scores.csv)
- Clustering of the filtered data should align better with tissue morphology than clustering of the unprocessed data
- Input matrix dimensions must be preserved in checks: 53,812 pixel rows and 2,654 ion columns before filtering

## Limitations

- Tissue and background regions are user-defined via the lasso tool on the median pixel intensity image, so region selection is subjective
- No software version specified for SMQVP and no license information stated in the repository documentation; no changelog available
- The noise-score threshold of 90 was reported for the AFADESI mouse brain dataset and may not transfer to other instruments or tissue types without re-evaluation

## Evidence

- [abstract] SMQVP incorporates eight comprehensive quality visualization and evaluation modules, including background consistency assessments, noise ion filtering, intensity distribution analyses: "SMQVP incorporates eight comprehensive quality visualization and evaluation modules, including background consistency assessments, noise ion filtering, intensity distribution analyses"
- [readme] Use the lasso tool to delineate background and tissue regions on the interaction image: "**Background and Tissue Region Selection**: Use the lasso tool to delineate background and tissue regions on the interaction image."
- [readme] The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode. The data includes 53,812 pixels and 2,654 ion peaks.: "The example data used in this study is spatial metabolomics data of a mouse brain acquired using the AFAD-ESI platform in positive ion mode. The data includes 53,812 pixels and 2,654 ion peaks."
- [results] QVP3 analysis of the AFADESI mouse brain dataset showed that 37% of the detected ions were enriched in the tissue.: [evidence span withheld — anchor status: not_found]
- [results] SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality.: "SMQVP has been demonstrated to effectively improve clustering results by removing noise signals and enhancing data quality."
