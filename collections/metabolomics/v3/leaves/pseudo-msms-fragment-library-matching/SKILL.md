---
name: pseudo-msms-fragment-library-matching
description: Use when annotating features from AIF LC–MS/MS data by ranking library
  candidates per feature using m/z error and fragment matching against pseudo-MS/MS
  spectra.
when_to_use_negative:
- Input is targeted MS/MS data with true precursor-selected fragmentation spectra
  — spectral matching to MS/MS libraries (METLIN, MassBank, GNPS) is more appropriate.
- Data were acquired on a DIA scheme other than AIF (e.g., SWATH), which MetaboAnnotatoR
  does not currently support.
- The feature table was already fully annotated (e.g., by MS-DIAL) and no re-ranking
  of library candidates is needed.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnnotatoR
  role: Provides searchLib, compFrag, and rankScore functions that compute the composite
    score and rank candidate metabolites per feature
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Language in which the scoring and ranking workflow is implemented and executed
- name: MS-DIAL
  role: State-of-the-art AIF annotation baseline used to benchmark the ranked candidate
    results
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:1180052527d939ae3ef5c39b0b8ac4cb81f0f55036a8ddaa9216d27e95796521
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MS-DIAL
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# pseudo-msms-fragment-library-matching

## Summary

Scores and ranks candidate metabolites per LC–MS feature by combining a bounded reciprocal m/z error term with a pseudo-MS/MS fragment-matching score, downweighting AIF-only fragments, as implemented in MetaboAnnotatoR for AIF untargeted metabolomics annotation.

## When to use

Use when annotating features from AIF LC–MS/MS data by ranking library candidates per feature using m/z error and fragment matching against pseudo-MS/MS spectra.

## When NOT to use

- Input is targeted MS/MS data with true precursor-selected fragmentation spectra — spectral matching to MS/MS libraries (METLIN, MassBank, GNPS) is more appropriate.
- Data were acquired on a DIA scheme other than AIF (e.g., SWATH), which MetaboAnnotatoR does not currently support.
- The feature table was already fully annotated (e.g., by MS-DIAL) and no re-ranking of library candidates is needed.

## Inputs

- rCandidates object with candidate formulas and library fragments per feature (MetaboAnnotatoR package data)
- Pseudo-MS/MS spectrum per feature reconstructed from AIF scans (e.g., via xcms and RAMClustR processing)
- Reference metabolite example (acetaminophen) from MetaboAnnotatoR package data
- Fragment library (.txt or .msp), default lipidomics/nonlipid libraries from the MetaboAnnotatoR GitHub

## Outputs

- Ranked candidate metabolite list per feature with composite score S and subscores (Smz, Sma)
- High-rank annotations (ranks 1–5) of metabolites, isotopologues, adducts, and in-source fragments

## How to apply

Load the rCandidates object containing candidate formulas and their library fragments, plus a reference (e.g., acetaminophen) from MetaboAnnotatoR package data. Compute the composite score S = w_mz·min(1/E,1) + w_ma·S_ma (default weights w_mz = w_ma = 0.5), where S_ma sums the library occurrence scores of fragments matched to the feature's pseudo-MS/MS spectrum and halves the contribution of fragments matched only in the AIF scan because they are less specific. Rank candidate metabolites per feature by descending S, then cross-check against MetaboAnnotatoR's compFrag, rankScore, and searchLib functions. The rationale is that correlation-based parent–fragment linking alone yields incomplete pseudo-MS/MS spectra, so fragment-library matching adds specificity beyond co-elution.

## Related tools

- **MetaboAnnotatoR** (Provides searchLib, compFrag, and rankScore functions that compute the composite score and rank candidate metabolites per feature) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Language in which the scoring and ranking workflow is implemented and executed)
- **MS-DIAL** (State-of-the-art AIF annotation baseline used to benchmark the ranked candidate results)

## Examples

```
R> install_github("gggraca/MetaboAnnotatoR", dependencies = TRUE); R> candidates <- searchLib(feature, rCandidates, lib = nonlipidLib)
```

## Evaluation signals

- Computed rank scores and candidate ordering match MetaboAnnotatoR's rankScore/searchLib output on the acetaminophen example data.
- Fragments matched only in the AIF scan contribute half weight relative to pseudo-MS/MS-matched fragments.
- Weights default to w_mz = w_ma = 0.5 and the m/z term is bounded as min(1/E,1).
- Precision and recall for high-rank scores (1–5) fall in the reported 82–92% and 82–85% ranges on the human serum datasets.
- Results equal or outperform MS-DIAL on the same 599 expert-annotated serum features.

## Limitations

- AIF-only fragments are less specific and are downweighted by a factor of two, so candidates matching only via AIF scans rank lower.
- Recall can be low (10–88%) for non-serum matrices dominated by nonlipid metabolites due to the small size of the nonlipid fragment library.
- Only AIF DIA is supported; SWATH and other schemes are not implemented, and Orbitrap/higher mass accuracy acquisition was not tested.
- Fragments not captured by the pseudo-MS/MS spectrum can only be searched in AIF scans with reduced specificity.

## Evidence

- [full_text] The scoring combines a bounded reciprocal m/z error term (Smz = min(1/E,1)) with a fragmentation score Sma summing occurrence scores of fragments matched to pseudo-MS/MS: [evidence span withheld — anchor status: not_found]
- [discussion] good precision (82–92%) and recall (82–85%) values were obtained for human serum datasets for high-rank scores (1–5).: "good precision (82–92%) and recall (82–85%) values were obtained for human serum datasets for high-rank scores (1–5)."
- [discussion] the ability for peaks not captured by pseudo-MS/MS to be searched in AIF scans: "the ability for peaks not captured by pseudo-MS/MS to be searched in AIF scans"
- [abstract] combining correlation-based parent–fragment linking with molecular fragment matching: "combining correlation-based
parent–fragment linking with molecular fragment matching"
- [discussion] Although only AIF is currently supported, future developments might expand MetaboAnnotatoR's application to other DIA schemes, such as SWATH.: "Although only AIF is currently supported, future developments might expand MetaboAnnotatoR’s application to other DIA schemes, such as SWATH."
