---
name: metabolomics-feature-correspondence-mapping
description: Use when you need to cross-validate an untargeted metabolomics quantification
  pipeline (e.g., eRah deconvolution areas) against an alternative peak-picking feature
  table (e.
when_to_use_negative:
- When you only need to run a single tool's pipeline end-to-end and have no alternative
  tool's feature table to compare against.
- When the feature tables are not row-matched to the same sample set and compound
  identifiers (cross-sample or cross-compound alignment is required first, e.g., via
  eRah's alignComp step).
- When the input is raw chromatogram/mass-spectra data rather than already-extracted
  area/intensity tables.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: erah
  role: Produces the deconvolved compound areas (elution-profile areas) used as the
    regression response
  repo: https://github.com/xdomingoal/erah-devel
- name: XCMS
  role: Provides the alternative centWave mzRT ion-peak areas for comparison
- name: MetAlign
  role: Provides the alternative MetAlign peak areas for comparison
- name: R
  role: Implementation language for eRah and the regression/assembly of the R² comparison
    table
  repo: http://CRAN.R-project.org/package=erah
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:ab608f63b8116936b191d265b705d8745f1cb84c69c83374d673c77519b61450
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xdomingoal/erah-devel
  tools:
  - MetAlign
  - R
  - XCMS
  - erah
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolomics-feature-correspondence-mapping

## Summary

Maps quantification values from an untargeted metabolomics tool (e.g., eRah's deconvolved compound areas) to those of other peak-picking tools (e.g., XCMS centWave, MetAlign) across matched samples to quantify agreement. Used here to regress eRah deconvolution-derived areas on selective mzRT ion-feature areas/intensities and compute per-compound R².

## When to use

Use when you need to cross-validate an untargeted metabolomics quantification pipeline (e.g., eRah deconvolution areas) against an alternative peak-picking feature table (e.g., XCMS centWave mzRT features or MetAlign peak areas) for the same samples and compounds, computing per-compound agreement (R²) as evidence of methodological consistency.

## When NOT to use

- When you only need to run a single tool's pipeline end-to-end and have no alternative tool's feature table to compare against.
- When the feature tables are not row-matched to the same sample set and compound identifiers (cross-sample or cross-compound alignment is required first, e.g., via eRah's alignComp step).
- When the input is raw chromatogram/mass-spectra data rather than already-extracted area/intensity tables.

## Inputs

- eRah deconvolved compound area/intensity table (relative concentrations per compound per sample)
- XCMS centWave feature table (mzRT ion-peak areas) for the same samples
- MetAlign peak area/intensity table for the same samples
- List of target compounds (e.g., the 33 compounds in Table 1) with identifiers linking to AlignID/feature IDs

## Outputs

- Per-compound R² table comparing eRah vs XCMS and eRah vs MetAlign quantification across samples
- Cross-tool correspondence map linking compound identities to feature IDs in each pipeline

## How to apply

Load the sample-matched area/intensity tables from each pipeline (eRah's deconvolved elution-profile areas, XCMS centWave quantitative ion-peak areas with parameters such as ppm=15 and peakwidth=c(1,5), and MetAlign areas) and restrict them to the compounds of interest, matching rows by compound name and AlignID/feature identifier. For each compound, regress the eRah values against each alternative tool's values across the same samples (e.g., 25 serum samples from 11 HIAE adolescents and 14 matched controls in the HIAE GC–TOF-MS study) and extract the coefficient of determination (R²) per comparison. Assemble the per-compound R² values into a table keyed by compound identifiers. High R² (article reports R² > 0.90 for most compounds in Table 1) indicates that deconvolved compound elution-profile areas are in excellent linear correlation with selective mzRT ion features, validating the deconvolution-based quantification.

## Related tools

- **erah** (Produces the deconvolved compound areas (elution-profile areas) used as the regression response) — https://github.com/xdomingoal/erah-devel
- **XCMS** (Provides the alternative centWave mzRT ion-peak areas for comparison)
- **MetAlign** (Provides the alternative MetAlign peak areas for comparison)
- **R** (Implementation language for eRah and the regression/assembly of the R² comparison table) — http://CRAN.R-project.org/package=erah

## Examples

```
Rscript r2_comparison_analysis.R --input eRah_areas.csv --xcms xcms_areas.csv --metalign metalign_areas.csv --compounds table1_compounds.csv --output r2_comparison_results.csv
```

## Evaluation signals

- Every target compound (e.g., all 33 Table 1 compounds) has an R² value for both eRah-vs-XCMS and eRah-vs-MetAlign comparisons.
- All R² values fall within the valid range [0, 1].
- Most compounds exhibit R² > 0.90, reproducing the article's reported excellent linear correlation between eRah and XCMS/MetAlign quantitative results.
- Sample counts match across the compared tables (e.g., 25 serum samples: 11 HIAE and 14 controls).
- Row-matching by compound name/AlignID is verified so regression pairs are not misaligned by feature-ID mismatches.

## Limitations

- Correlation is high only for compounds whose selective mzRT features are well-matched to the deconvolved spectra; co-eluted or very-low-abundance compounds may show weaker correspondence.
- Requires that the compared pipelines were run on the same raw data and sample set; differences in parameters (e.g., centWave ppm=15, peakwidth=c(1,5)) or version differences can shift feature identities.
- R² measures linear agreement in relative concentration/area only; it does not validate compound identification correctness by itself.
- eRah must be run after alignment/recMissComp steps so that the deconvolved compound table is aligned across samples.

## Evidence

- [abstract] Table 1 comparison basis: "The quantitative results of eRah are compared to centWave, the peak-picking algorithm implemented in the widely used XCMS package, MetAlign, and ChromaTOF software."
- [results] High agreement found: [evidence span withheld — anchor status: not_found]
- [abstract] eRah output schema: "e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
- [discussion] Supporting files location: "This material is available free of charge via the Internet at http://pubs.acs.org/."
- [readme] eRah repo / install: [evidence span withheld — anchor status: external_doc]
