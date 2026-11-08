---
name: cross-tool-peak-area-correlation-analysis
description: Use when two or more GC–MS peak-picking or deconvolution tools (e.g.
  eRah, XCMS centWave, MetAlign) have produced per-sample quantification tables and
  you need to quantify their agreement by correlating peak areas/intensities per compound
  across samples.
when_to_use_negative:
- Input is already a single tool's peak table and no second tool's quantification
  exists to compare against
- The goal is to evaluate identification accuracy (spectral match scores, library
  hits) rather than quantitative agreement of peak areas
- Samples or compound sets are not shared across tools, so no per-sample per-compound
  pairing is possible
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: eRah
  role: Produces the deconvolved compound area/intensity tables that serve as the
    reference quantitative values to be correlated against other tools' peak-picking
    outputs
  repo: http://CRAN.R-project.org/package=erah
- name: XCMS
  role: Provides centWave-based mzRT ion-feature quantification (ppm=15, peakwidth=c(1,5))
    used as one comparison side of the regression
- name: MetAlign
  role: Provides peak area/intensity tables used as the other comparison side of the
    regression
- name: R
  role: Implementation environment for loading the tables, performing the per-compound
    linear regressions and assembling the R² table
  repo: http://CRAN.R-project.org/package=erah
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:61a57f76f99ae8806569371bbcdef8c67b10e6b044e6a52f9f8cc05bf05080cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetAlign
  - R
  - XCMS
  - eRah
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-tool-peak-area-correlation-analysis

## Summary

Regress eRah's deconvolved compound elution-profile areas (or intensities) against the quantitative mzRT ion-peak features output by XCMS (centWave) and MetAlign across the same samples, extracting per-compound coefficients of determination (R²) to assess cross-tool quantitative agreement in GC–MS untargeted metabolomics.

## When to use

Use when two or more GC–MS peak-picking or deconvolution tools (e.g. eRah, XCMS centWave, MetAlign) have produced per-sample quantification tables and you need to quantify their agreement by correlating peak areas/intensities per compound across samples.

## When NOT to use

- Input is already a single tool's peak table and no second tool's quantification exists to compare against
- The goal is to evaluate identification accuracy (spectral match scores, library hits) rather than quantitative agreement of peak areas
- Samples or compound sets are not shared across tools, so no per-sample per-compound pairing is possible

## Inputs

- eRah deconvolved compound table with compound names, matching scores and integrated area of compounds for each sample
- XCMS centWave quantitative mzRT ion-feature area/intensity table per sample
- MetAlign peak area/intensity table per sample
- Compound list shared across tools (e.g. the 33 compounds listed in Table 1 of the article)

## Outputs

- Per-compound R² values for each tool comparison (e.g. eRah vs XCMS, eRah vs MetAlign) with compound identifiers
- r2_comparison_results.csv

## How to apply

Restrict each tool's output table (eRah's deconvolved area/intensity table, XCMS centWave mzRT feature areas computed with ppm=15 and peakwidth=c(1,5), and MetAlign peak areas) to the same set of compounds and the same samples, matching rows by compound name and AlignID/feature identifiers. For each compound, fit a linear regression of the eRah area/intensity on each comparison tool's area across samples and extract the coefficient of determination (R²) for each comparison. The paper's HIAE serum GC–TOF-MS analysis (25 serum samples from 11 HIAE adolescents and 14 matched controls) demonstrated an excellent linear correlation (R²>0.90) for most compounds, even for coeluted or low relative concentration compounds where eRah's deconvolved profiles were compared to selective mzRT features. Assemble the per-compound R² values for each tool pair into a table with compound identifiers (e.g. r2_comparison_results.csv).

## Related tools

- **eRah** (Produces the deconvolved compound area/intensity tables that serve as the reference quantitative values to be correlated against other tools' peak-picking outputs) — http://CRAN.R-project.org/package=erah
- **XCMS** (Provides centWave-based mzRT ion-feature quantification (ppm=15, peakwidth=c(1,5)) used as one comparison side of the regression)
- **MetAlign** (Provides peak area/intensity tables used as the other comparison side of the regression)
- **R** (Implementation environment for loading the tables, performing the per-compound linear regressions and assembling the R² table) — http://CRAN.R-project.org/package=erah

## Examples

```
Rscript r2_comparison.R --data-dir supporting_files/ --out r2_comparison_results.csv
```

## Evaluation signals

- Verify the output table contains an R² entry for each of the target compounds (e.g. the 33 Table 1 compounds) under each tool comparison (eRah vs XCMS and eRah vs MetAlign)
- Check that every R² value lies in the valid range [0, 1]
- Spot-check that high agreement reported in the article (R²>0.90 for most compounds) is reproduced on the HIAE serum GC–TOF-MS data
- Confirm row counts and compound identifiers match across the three input tables before regression
- Compare per-compound R² values against expected qualitative trends: coeluted or low-abundance compounds should still show high correlation between deconvolved profiles and selective mzRT features

## Limitations

- The validation scope is limited to serum samples analyzed by GC–TOF MS; extension to other matrices or instruments is not established in the article
- Detailed methods for the HIAE comparison processing are deferred to Supporting Files 1–4 hosted externally at pubs.acs.org rather than included in the main text
- No explicit version numbers are stated for eRah, XCMS, MetAlign, or ChromaTOF, so reproducibility depends on unspecified software versions
- eRah's deconvolved compound values represent integrated area of the compound spectrum, not a single ion trace, so direct comparison with fragment-ion-based mzRT features depends on selective ion choice

## Evidence

- [abstract] R²>0.90 for most compounds: [evidence span withheld — anchor status: not_found]
- [abstract] eRah outputs a table with compound names, matching scores and the integrated area of compounds for each sample: "e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
- [abstract] comparing eRah to XCMS centWave and MetAlign: "The quantitative results of eRah are compared to centWave, the peak-picking algorithm implemented in the widely used XCMS package, MetAlign, and ChromaTOF software."
- [methods] XCMS centWave parameters used in the article: [evidence span withheld — anchor status: not_found]
- [discussion] scope limited to GC–TOF MS serum samples: "The comparative analysis of serum samples by GC–TOF MS provided test data"
- [abstract] era h is a free computational tool written in the open language R: "a free computational tool written in the open language R composed of five core functions"
- [readme] installation from CRAN: [evidence span withheld — anchor status: not_found]
