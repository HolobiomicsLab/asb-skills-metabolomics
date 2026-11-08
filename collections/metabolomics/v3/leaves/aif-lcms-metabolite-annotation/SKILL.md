---
name: aif-lcms-metabolite-annotation
description: Use when annotating features from centroid-mode LC-MS AIF chromatograms
  (e.g., xcms/RAMClustR output) against fragment libraries, especially when isotopologues,
  in-source fragments, and adducts must be labeled and only a statistically significant
  subset of features needs annotation.
when_to_use_negative:
- Data were acquired with DIA schemes other than AIF (e.g., SWATH) — only AIF is currently
  supported by MetaboAnnotatoR.
- The goal is exhaustive untargeted annotation of all detected features regardless
  of statistical significance — the workflow deliberately focuses on a subset of features
  and may miss low-intensity ones outside that subset.
- High-confidence MS/MS spectral matching on high mass accuracy instruments (e.g.,
  Orbitrap) is required — although reportedly possible, this has not been tested/validated
  in the paper.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: MetaboAnnotatoR
  role: Primary R package implementing correlation-based parent–fragment linking and
    molecular fragment matching for AIF annotation
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Runtime environment for installing and running the MetaboAnnotatoR package
- name: MS-DIAL
  role: State-of-the-art baseline used to benchmark annotation performance (equal
    or outperformed)
- name: RAMClustR
  role: Used upstream to process chromatograms and reconstruct pseudo-MS/MS features
    fed into MetaboAnnotatoR
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:edf136188b7fa49d38ba36ccfb5ced95251fbe71a29b5a2e2761fc5feed6c39a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MS-DIAL
  - MetaboAnnotatoR
  - R
  - RAMClustR
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# aif-lcms-metabolite-annotation

## Summary

Automated annotation of isotopologues, adducts, and in-source fragments in untargeted all-ion fragmentation (AIF) LC–MS metabolomics data by combining correlation-based parent–fragment linking with molecular fragment matching, via the open-source MetaboAnnotatoR R package.

## When to use

Use when annotating features from centroid-mode LC-MS AIF chromatograms (e.g., xcms/RAMClustR output) against fragment libraries, especially when isotopologues, in-source fragments, and adducts must be labeled and only a statistically significant subset of features needs annotation.

## When NOT to use

- Data were acquired with DIA schemes other than AIF (e.g., SWATH) — only AIF is currently supported by MetaboAnnotatoR.
- The goal is exhaustive untargeted annotation of all detected features regardless of statistical significance — the workflow deliberately focuses on a subset of features and may miss low-intensity ones outside that subset.
- High-confidence MS/MS spectral matching on high mass accuracy instruments (e.g., Orbitrap) is required — although reportedly possible, this has not been tested/validated in the paper.

## Inputs

- Raw LC-MS AIF chromatograms in centroid mode
- Feature tables from XCMS or similar peak-picking software (e.g., xset/RC objects, targetTable.csv)
- Fragment/molecular fragment libraries (.txt or .msp format; e.g., LipidPos library, LipidBlast-derived libraries)
- Expert manual annotations for validation datasets (human serum, adipose tissue ESI+/ESI−, MTBLS666 amniotic fluid, MTBLS816 urine)

## Outputs

- Annotated feature table with isotopologue (M+0–M+3), adduct, and in-source fragment assignments and rank scores
- Per-dataset precision and recall metrics (e.g., metabolite_annotation_results.csv)

## How to apply

Install MetaboAnnotatoR from GitHub (devtools::install_github('gggraca/MetaboAnnotatoR')) and supply raw LC-MS AIF chromatograms acquired or transformed in centroid mode, together with features from XCMS or similar preprocessing software. The workflow combines correlation-based parent–fragment linking with molecular fragment matching against fragment libraries (e.g., LipidPos/LipidBlast-derived; customizable by importing external .txt and .msp files). For each target feature, the checkIsotope function (checkIsotope(fmz, frt, spec, mztol)) extracts MS1 EICs within the same retention-time window, computes peak-shape correlations with candidates at m/z −1, −2, −3 Da, and iteratively assigns the monoisotopic (M+0) peak as the highest-intensity first peak of the correlated series, labeling the feature M+0 through M+3. The workflow focuses on a subset of statistically significant features rather than annotating the full dataset, saving time relative to untargeted tools like MS-DIAL. Aggregate per-feature annotations and evaluate precision and recall against expert manual annotations; on three human serum datasets (599 features) expect precision 82–92% and recall 82–85% for rank scores 1–5.

## Related tools

- **MetaboAnnotatoR** (Primary R package implementing correlation-based parent–fragment linking and molecular fragment matching for AIF annotation) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Runtime environment for installing and running the MetaboAnnotatoR package)
- **MS-DIAL** (State-of-the-art baseline used to benchmark annotation performance (equal or outperformed))
- **RAMClustR** (Used upstream to process chromatograms and reconstruct pseudo-MS/MS features fed into MetaboAnnotatoR)

## Examples

```
library(devtools); install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE, INSTALL_opts = c('--no-multiarch', '--no-lock'))
```

## Evaluation signals

- Precision and recall on the three human serum datasets (599 manually annotated features) fall within 82–92% (precision) and 82–85% (recall) for high-rank scores (1–5).
- Per-dataset precision/recall on the four external validation datasets (adipose tissue ESI+/ESI−, MTBLS666 amniotic fluid, MTBLS816 urine) match Table 2 ranges: precision 60–89%, recall 10–88%.
- checkIsotope-assigned isotopologue labels are consistent with correlated intensity ordering across −1 Da EIC steps (monoisotopic candidate must have highest intensity at the earliest eluting apex among correlated EICs); ambiguous features are flagged for manual review rather than forced.
- Results equal or outperform MS-DIAL annotations on the same AIF datasets.
- Annotation ranks concentrate in scores 1–5 for correct assignments, consistent with the reported high-rank-score performance.

## Limitations

- Variable precision (60–89%) and low recall (10–88%) on non-serum matrices, attributable to the small size of the fragment libraries, in particular the nonlipid library.
- Only AIF is supported; other DIA schemes such as SWATH are not implemented or validated.
- Annotation of AIF datasets from higher mass accuracy instruments (e.g., Orbitraps) is possible but was not tested.
- Installation may fail due to dependency conflicts (e.g., 'mzR has been built against a different Rcpp version'); workarounds include Sys.setenv(R_REMOTES_NO_ERRORS_FROM_WARNINGS='true') or INSTALL_opts c('--no-multiarch', '--no-lock').
- Peak shape matching/correlation deconvolution can yield incomplete pseudo-MS/MS spectra for co-eluting compounds with similar chemical groups producing nonspecific fragments.

## Evidence

- [abstract] combining correlation-based parent–fragment linking with molecular fragment matching: "combining correlation-based
parent–fragment linking with molecular fragment matching"
- [abstract] Precision and recall values of 82–92% and 82–85%: "Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found
in the highest-rank scores (1–5)"
- [readme] Installation requires devtools and centroid-mode input: "It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode."
- [results] checkIsotope monoisotopic identification mechanism: [evidence span withheld — anchor status: not_found]
- [discussion] Low recall on non-serum matrices due to library size: "These showed modest-to-good (60–88%) precision but low recall due to the small size of the fragment libraries, in particular, the nonlipid library."
- [discussion] Only AIF supported; SWATH not implemented: "Although only AIF is currently supported, future developments might expand MetaboAnnotatoR’s application to other DIA schemes, such as SWATH."
