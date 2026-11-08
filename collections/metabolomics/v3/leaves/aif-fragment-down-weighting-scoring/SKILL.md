---
name: aif-fragment-down-weighting-scoring
description: Use when ranking candidate metabolite annotations for features from all-ion
  fragmentation (AIF) LC–MS data, where each candidate must be scored by combining
  m/z error with pseudo-MS/MS fragment matching and AIF-only matches must be penalized.
when_to_use_negative:
- When the goal is fragmentation-spectrum matching against curated MS/MS spectral
  databases (METLIN, MassBank, GNPS) from true DDA MS/MS rather than AIF pseudo-MS/MS
  spectra
- When input data were not acquired as all-ion fragmentation LC–MS/MS — only AIF is
  currently supported by MetaboAnnotatoR
- When the input is already an annotated feature table; this skill scores and ranks
  candidate annotations, it does not create the feature list
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboAnnotatoR
  role: Open-source R package providing the compFrag, rankScore, and searchLib functions
    that implement the composite AIF fragment-down-weighted scoring and candidate
    ranking
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Runtime environment in which the scoring/ranking is implemented and validated
    against package data
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: MS-DIAL
  role: State-of-the-art AIF annotation software used as an external performance benchmark
    for the scored rankings
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:ef2b56132bf096e689c510db464bb5a41e4bd885e3321aec59e5b0d64ea1dab4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MS-DIAL
  - MetaboAnnotatoR
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# aif-fragment-down-weighting-scoring

## Summary

Composite candidate-scoring mechanism from MetaboAnnotatoR that ranks library metabolite matches per LC–MS AIF feature by combining a bounded reciprocal m/z error term with a fragment-occurrence score in which fragments matched only in the AIF scan (rather than the pseudo-MS/MS spectrum) are down-weighted, reflecting their lower specificity.

## When to use

Use when ranking candidate metabolite annotations for features from all-ion fragmentation (AIF) LC–MS data, where each candidate must be scored by combining m/z error with pseudo-MS/MS fragment matching and AIF-only matches must be penalized.

## When NOT to use

- When the goal is fragmentation-spectrum matching against curated MS/MS spectral databases (METLIN, MassBank, GNPS) from true DDA MS/MS rather than AIF pseudo-MS/MS spectra
- When input data were not acquired as all-ion fragmentation LC–MS/MS — only AIF is currently supported by MetaboAnnotatoR
- When the input is already an annotated feature table; this skill scores and ranks candidate annotations, it does not create the feature list

## Inputs

- rCandidates object: candidate formulas and associated library fragments per LC–MS AIF feature
- Pseudo-MS/MS spectrum per feature (reconstructed from AIF data via correlation-based parent–fragment linking)
- Metabolite fragment library (MetaboAnnotatoR default libraries or external .txt/.msp imports)
- Reference metabolite entries (e.g., the acetaminophen reference from MetaboAnnotatoR package data for validation)

## Outputs

- Ranked candidate metabolite list per feature, sorted by descending composite score S
- Per-feature rank scores combining m/z error (min(1/E,1)) and fragment match subscore S_ma

## How to apply

For each feature, compute the composite score S = w_mz·min(1/E,1) + w_ma·S_ma, where E is the m/z error of the candidate and S_ma is obtained by summing library fragment occurrence scores of fragments matched to the feature's pseudo-MS/MS spectrum reconstructed from AIF data. Fragments matched only in the low-energy-adjacent AIF scan are down-weighted by a factor of two because they are nonspecific and may arise from co-eluting compounds. Use default subscore weights w_mz = w_ma = 0.5 unless validation suggests otherwise, then sort candidate metabolites per feature by descending S to produce the ranked annotation list. Cross-check the reimplementation against the package's compFrag, rankScore, and searchLib functions using the acetaminophen example from the MetaboAnnotatoR package data, and validate rank-score quality using precision and recall for high-rank scores (1–5) against expert annotations.

## Related tools

- **MetaboAnnotatoR** (Open-source R package providing the compFrag, rankScore, and searchLib functions that implement the composite AIF fragment-down-weighted scoring and candidate ranking) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Runtime environment in which the scoring/ranking is implemented and validated against package data) — https://github.com/gggraca/MetaboAnnotatoR
- **MS-DIAL** (State-of-the-art AIF annotation software used as an external performance benchmark for the scored rankings)

## Examples

```
library(MetaboAnnotatoR); data(rCandidates); data(acetaminophen); ranked <- rankScore(rCandidates, acetaminophen)
```

## Evaluation signals

- Computed rank scores and candidate ordering for the acetaminophen example match the values returned by the package's rankScore/searchLib reference implementation (expert review of scored candidate agreement)
- Fragments matched only in the AIF spectrum contribute exactly half weight relative to fragments matched in the pseudo-MS/MS spectrum
- With default weights, the m/z-error term is bounded: min(1/E,1) ≤ 1, and S = 0.5·min(1/E,1) + 0.5·S_ma
- Precision and recall for high-rank scores (1–5) against expert-annotated serum features fall in the ranges reported for the method (82–92% precision, 82–85% recall on human serum datasets)

## Limitations

- Fragments appearing only in the AIF scan are less specific because co-eluting compounds with similar chemical groups produce nonspecific fragment ions, so down-weighting reduces but does not eliminate false matches from incomplete pseudo-MS/MS spectra
- Rank-score quality depends on fragment library coverage; low recall (10–88%) was observed for non-human-serum matrices, attributed to the small nonlipid fragment library
- Precision/recall trade-off is defined only for high-rank scores (1–5); candidates ranked lower have not been validated

## Evidence

- [abstract] combining correlation-based parent–fragment linking with molecular fragment matching: "combining correlation-based parent–fragment linking with molecular fragment matching"
- [abstract] Precision and recall for high-rank scores: "Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found in the highest-rank scores (1–5)"
- [discussion] AIF-only fragment search rationale: "the ability for peaks not captured by pseudo-MS/MS to be searched in AIF scans"
- [readme] README: centroid-mode AIF input requirement: "It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode."
- [discussion] Fragment library customization: "these can be customized by the user, for instance, by importing external libraries as .txt and .msp files."
