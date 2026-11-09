---
name: linear-regression-r2-computation
description: Use when two GC–MS preprocessing tools have produced per-sample compound/feature
  quantification tables for the same samples and compounds, and you need a numeric
  per-compound agreement metric (R²) to validate that a new method (e.g., eRah) reproduces
  established quantification.
when_to_use_negative:
- Input is a single preprocessing tool's feature table with no second platform's quantification
  for the same samples/compounds — there is nothing to correlate against.
- The compounds in the two tables cannot be matched by name or identifier (e.g., eRah
  AlignID vs XCMS mzRT features have no crosswalk), since per-compound pairing is
  required before regression.
- The goal is peak picking or deconvolution itself rather than post-hoc quantification
  comparison; use eRah's deconvolveComp/alignComp pipeline or XCMS centWave first.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: erah
  role: Source of the deconvolved compound quantification (integrated areas) regressed
    against the other tools' values
  repo: http://CRAN.R-project.org/package=erah
- name: XCMS
  role: Provides the comparison quantitative mzRT ion-feature areas (centWave peak
    picking) used as regression predictors
- name: MetAlign
  role: Provides the alternative comparison peak area/intensity table regressed against
    eRah areas
- name: R
  role: Language in which eRah runs and in which the per-compound linear regressions
    and R² extraction are performed
  repo: http://CRAN.R-project.org/package=erah
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.6b02927
    title: 'eRah: A Computational Tool Integrating Spectral Deconvolution and Alignment
      with Quantification and Identification of Metabolites in GC/MS-Based Metabolomics'
  source_content_hash: sha256:bdd28a1a11ae1ebe7f3630b58ae66f603d712a2b91e105724b8c89da9c1cc91a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetAlign
  - R
  - XCMS
  - erah
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# linear-regression-r2-computation

## Summary

Compute per-compound coefficients of determination (R²) by linearly regressing one platform's quantitative metabolite values (e.g., eRah's integrated deconvolved elution-profile areas) against another's (e.g., XCMS centWave or MetAlign ion-feature areas) across matched serum samples, to quantify cross-tool quantification agreement.

## When to use

Use when two GC–MS preprocessing tools have produced per-sample compound/feature quantification tables for the same samples and compounds, and you need a numeric per-compound agreement metric (R²) to validate that a new method (e.g., eRah) reproduces established quantification.

## When NOT to use

- Input is a single preprocessing tool's feature table with no second platform's quantification for the same samples/compounds — there is nothing to correlate against.
- The compounds in the two tables cannot be matched by name or identifier (e.g., eRah AlignID vs XCMS mzRT features have no crosswalk), since per-compound pairing is required before regression.
- The goal is peak picking or deconvolution itself rather than post-hoc quantification comparison; use eRah's deconvolveComp/alignComp pipeline or XCMS centWave first.

## Inputs

- eRah quantification table: compound names, matching scores, and integrated deconvolved compound areas per sample (obtainable via dataList(ex) / alignList(ex))
- XCMS centWave quantitative mzRT ion-feature table (e.g., ppm=15, peakwidth=c(1,5)) for the same samples
- MetAlign peak area/intensity table for the same samples
- Compound mapping/identifier list restricting analysis to shared compounds (e.g., the 33 Table 1 compounds)
- Sample metadata matching rows across tables (25 serum samples: 11 HIAE adolescents, 14 matched controls)

## Outputs

- Per-compound R² table for eRah-vs-XCMS and eRah-vs-MetAlign regressions (r2_comparison_results.csv)
- Per-compound regression fits (coefficient of determination per compound, per comparison)

## How to apply

Load the paired quantification tables: eRah's output table (compound names, matching scores, and integrated area of the compound for each sample) and the XCMS centWave or MetAlign peak-area/intensity tables for the same samples (here, 25 serum samples from 11 HIAE adolescents and 14 matched controls). Restrict both tables to the shared compound list (the 33 compounds in Table 1 of the eRah paper), matching rows by compound name and by the tool-specific identifiers (eRah AlignID vs XCMS/MetAlign mzRT features). For each compound, fit an ordinary least-squares regression of the eRah area/intensity values on the comparison tool's area values across the 25 samples, extracting R² for each comparison (eRah-vs-XCMS and eRah-vs-MetAlign). Assemble the per-compound R² values into a table with compound identifiers (e.g., r2_comparison_results.csv). Interpret high R² (>0.90 for most compounds in the paper) as evidence that the deconvolved compound areas correlate with the selective mzRT ion features; expect high agreement even for coeluted or low-relative-concentration compounds.

## Related tools

- **erah** (Source of the deconvolved compound quantification (integrated areas) regressed against the other tools' values) — http://CRAN.R-project.org/package=erah
- **XCMS** (Provides the comparison quantitative mzRT ion-feature areas (centWave peak picking) used as regression predictors)
- **MetAlign** (Provides the alternative comparison peak area/intensity table regressed against eRah areas)
- **R** (Language in which eRah runs and in which the per-compound linear regressions and R² extraction are performed) — http://CRAN.R-project.org/package=erah

## Examples

```
Rscript -e 'r2 <- sapply(33:1, function(i) summary(lm(erah_tab[,i] ~ xcms_tab[,i]))$r.squared); write.csv(data.frame(compound=names, r2_xcms=r2), "r2_comparison_results.csv")'
```

## Evaluation signals

- r2_comparison_results.csv contains an R² entry for each of the 33 Table 1 compounds under both comparisons (eRah vs XCMS and eRah vs MetAlign).
- All R² values lie in the valid range [0, 1].
- Most compounds reproduce the article's reported excellent linear correlation (R² > 0.90) between eRah and XCMS/MetAlign quantitative results.
- Coeluted or low relative concentration compounds still show high correlation, consistent with the paper's finding.
- Rows are matched one-to-one across tables by compound name and AlignID/mzRT feature identifiers before regression.

## Limitations

- Validation scope in the source study was limited to serum samples analyzed by GC–TOF MS (25 samples from 11 HIAE adolescents and 14 matched controls).
- Detailed methods for the HIAE comparison processing are deferred to supporting methods and Supporting Files 1–4 hosted externally at pubs.acs.org, not in the main text.
- No specific version numbers were stated for eRah, XCMS, MetAlign, or ChromaTOF, so exact reproducibility may depend on software versions.
- Correlation quality depends on correct compound matching across tools; mismatched AlignID/mzRT feature identifiers will corrupt the regressions.

## Evidence

- [abstract] The quantitative results of eRah are compared to centWave, the peak-picking algorithm implemented in the widely used XCMS package, MetAlign, and ChromaTOF software.: "The quantitative results of eRah are compared to centWave, the peak-picking algorithm implemented in the widely used XCMS package, MetAlign, and ChromaTOF software."
- [abstract] eRah outputs a table with compound names, matching scores and the integrated area of compounds for each sample: "e Rah outputs a table with compound names, matching scores and the integrated area of compounds for each sample"
- [discussion] The comparative analysis of serum samples by GC–TOF MS provided test data: "The comparative analysis of serum samples by GC–TOF MS provided test data"
- [readme] Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics.: "Automated compound deconvolution, alignment across samples, and identification of metabolites by spectral library matching in Gas Chromatography - Mass spectrometry (GC-MS) untargeted metabolomics."
- [methods] The alignment list can be returned using: alignList(ex): [evidence span withheld — anchor status: not_found]
