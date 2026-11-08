---
name: mass-defect-mz-error-weighting
description: Use when annotating features from AIF LC-MS metabolomics data and you
  must score/rank candidate metabolites per feature against a fragment library, combining
  m/z accuracy with pseudo-MS/MS fragment-match specificity.
when_to_use_negative:
- Data were acquired with DIA schemes other than AIF (e.g., SWATH) — MetaboAnnotatoR
  only supports AIF.
- Features already have true targeted MS/MS spectra — direct spectral matching to
  METLIN/MassBank/GNPS libraries is more specific than pseudo-MS/MS-based scoring.
- The aim is fragment-library construction or pseudo-MS/MS reconstruction rather than
  candidate ranking — this skill only covers the scoring/ranking step.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnnotatoR
  role: Provides compFrag, rankScore, and searchLib functions that implement the composite
    m/z-error + fragmentation scoring and candidate ranking; supplies the acetaminophen
    example data for cross-checking.
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Runtime environment; the scoring and ranking workflow is implemented as an
    open-source R package.
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: XCMS
  role: Upstream feature detection producing the feature table (m/z, RT, intensities)
    whose candidates are scored; MetaboAnnotatoR can be applied directly to XCMS outputs.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:06ee1b9830ff3a1dda05116169a90aff16ae3ff92bd97b5b20f085ef4c5db286
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MetaboAnnotatoR
  - R
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mass-defect-mz-error-weighting

## Summary

Composite candidate-scoring scheme from MetaboAnnotatoR that ranks fragment-library matches per feature by combining a bounded reciprocal m/z error term (S_mz = min(1/E,1)) with a fragmentation match score (S_ma), weighting subscores (default w_mz = w_ma = 0.5) and downweighting fragments matched only in the AIF scan by a factor of two.

## When to use

Use when annotating features from AIF LC-MS metabolomics data and you must score/rank candidate metabolites per feature against a fragment library, combining m/z accuracy with pseudo-MS/MS fragment-match specificity.

## When NOT to use

- Data were acquired with DIA schemes other than AIF (e.g., SWATH) — MetaboAnnotatoR only supports AIF.
- Features already have true targeted MS/MS spectra — direct spectral matching to METLIN/MassBank/GNPS libraries is more specific than pseudo-MS/MS-based scoring.
- The aim is fragment-library construction or pseudo-MS/MS reconstruction rather than candidate ranking — this skill only covers the scoring/ranking step.

## Inputs

- rCandidates object (R) listing candidate formulas and their library fragments per feature
- pseudo-MS/MS spectrum per feature reconstructed from AIF data (e.g., via xcms + RamClustR)
- Feature m/z values from centroid-mode AIF LC-MS chromatograms
- MetaboAnnotatoR fragment library entries (default libraries or user-imported .txt/.msp files)
- Acetaminophen reference example from MetaboAnnotatoR package data (for validation)

## Outputs

- Composite match score S per feature-candidate pair (weighted sum of m/z-error and fragmentation subscores)
- Ranked candidate metabolite list per feature (descending S, top ranks 1-5 used for precision/recall assessment)

## How to apply

Load the rCandidates object containing candidate formulas and their fragments for each feature, plus the pseudo-MS/MS spectrum reconstructed from the AIF run. For each candidate, compute the composite score S = w_mz·min(1/E,1) + w_ma·S_ma, where E is the m/z error and S_ma sums library occurrence scores of fragments matched to the pseudo-MS/MS spectrum. Fragments matched only in the AIF scan (not in the pseudo-MS/MS) contribute at half weight because they are less specific to the parent. Use the default weights w_mz = w_ma = 0.5 so m/z accuracy and fragmentation evidence contribute equally, then rank candidates per feature by descending S. Cross-check the implementation against the package's compFrag, rankScore, and searchLib functions using the acetaminophen example shipped in the package data.

## Related tools

- **MetaboAnnotatoR** (Provides compFrag, rankScore, and searchLib functions that implement the composite m/z-error + fragmentation scoring and candidate ranking; supplies the acetaminophen example data for cross-checking.) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Runtime environment; the scoring and ranking workflow is implemented as an open-source R package.) — https://github.com/gggraca/MetaboAnnotatoR
- **XCMS** (Upstream feature detection producing the feature table (m/z, RT, intensities) whose candidates are scored; MetaboAnnotatoR can be applied directly to XCMS outputs.)

## Examples

```
install.packages("devtools", dependencies = TRUE); library(devtools); install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE)
```

## Evaluation signals

- Computed rank scores and candidate ordering for the acetaminophen example match the values returned by the package's rankScore/searchLib reference implementation.
- Each subscore is bounded: the m/z term equals min(1/E,1) ≤ 1, and AIF-only fragments contribute at exactly half their occurrence score.
- With default weights, S = 0.5·min(1/E,1) + 0.5·S_ma, so a perfect m/z and full pseudo-MS/MS match yields the maximum composite score.
- Candidates within each feature are sorted strictly by descending S, and top-rank (1-5) assignments reproduce the paper's precision of 82-92% and recall of 82-85% on the human serum benchmark datasets.
- Fragments matched only in the AIF scan (not captured in the pseudo-MS/MS) are visibly downweighted relative to pseudo-MS/MS-matched fragments in intermediate score breakdowns.

## Limitations

- Only AIF acquisition is supported; SWATH and other DIA schemes are not implemented or validated.
- Orbitrap/high-mass-accuracy data were not tested, so the bounded reciprocal m/z error term's calibration on higher-accuracy instruments is unknown.
- Recall is limited by fragment library size, particularly the small nonlipid library, which caused low recall (down to 10-88%) in non-serum matrices.
- Pseudo-MS/MS reconstruction can be incomplete for co-eluting compounds with similar chemical groups, degrading the fragmentation subscore.

## Evidence

- [abstract] combining correlation-based parent–fragment linking with molecular fragment matching: "combining correlation-based parent–fragment linking with molecular fragment matching"
- [abstract] rank-score precision/recall on top ranks: "Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found in the highest-rank scores (1–5)"
- [discussion] customizable fragment libraries affect S_ma: "these can be customized by the user, for instance, by importing external libraries as .txt and .msp files."
- [readme] centroid-mode AIF input requirement: "It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode."
