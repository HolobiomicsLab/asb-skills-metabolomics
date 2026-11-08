---
name: lc-ms-candidate-scoring-formula-implementation
description: Use when re-implementing or verifying how MetaboAnnotatoR computes per-feature
  match scores and ranks candidate metabolites from AIF pseudo-MS/MS fragment matches
  and m/z error, or when cross-checking rankScore/searchLib outputs.
when_to_use_negative:
- Input is a DIA scheme other than AIF (e.g. SWATH) — MetaboAnnotatoR only supports
  AIF, so pseudo-MS/MS inputs would not be produced
- The feature table already contains final annotations and only needs summarizing
  — no candidate scoring is required
- Data were acquired on higher-mass-accuracy instruments such as Orbitraps without
  validation, since the scoring was tested on the article's serum datasets, not Orbitrap
  data
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboAnnotatoR
  role: Provides compFrag, rankScore, and searchLib reference functions, the rCandidates
    and acetaminophen example data, and the fragment libraries used to compute and
    validate the composite score
  repo: https://github.com/gggraca/MetaboAnnotatoR
- name: R
  role: Implementation language in which the scoring formula is re-implemented and
    cross-checked against the package
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:abe39bab4ee325b8dc9272f49ccdb764d19f64582890117f0c3b71ebadac43e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gggraca/MetaboAnnotatoR
  tools:
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# lc-ms-candidate-scoring-formula-implementation

## Summary

Implements MetaboAnnotatoR's composite rank score that combines a bounded reciprocal m/z error term (S_mz = min(1/E, 1)) with a fragmentation-occurrence score (S_ma) over pseudo-MS/MS matched fragments, then ranks candidate metabolites per feature. Faithful replication of this scoring is needed to reproduce the package's ranked candidate lists.

## When to use

Use when re-implementing or verifying how MetaboAnnotatoR computes per-feature match scores and ranks candidate metabolites from AIF pseudo-MS/MS fragment matches and m/z error, or when cross-checking rankScore/searchLib outputs.

## When NOT to use

- Input is a DIA scheme other than AIF (e.g. SWATH) — MetaboAnnotatoR only supports AIF, so pseudo-MS/MS inputs would not be produced
- The feature table already contains final annotations and only needs summarizing — no candidate scoring is required
- Data were acquired on higher-mass-accuracy instruments such as Orbitraps without validation, since the scoring was tested on the article's serum datasets, not Orbitrap data

## Inputs

- rCandidates object (candidate formulas and fragment lists per feature) from MetaboAnnotatoR package data
- Acetaminophen reference entry from MetaboAnnotatoR package data
- Pseudo-MS/MS spectrum per feature reconstructed from AIF LC-MS data (via parent–fragment correlation linking)
- Fragment occurrence scores from the MetaboAnnotatoR fragment libraries (e.g. LipidBlast-derived lipid library, nonlipid library)

## Outputs

- Per-feature composite match score S for each candidate metabolite
- Ranked candidate list per feature ordered by descending S (rank 1–5 used for precision/recall evaluation)

## How to apply

Load the example rCandidates object and the acetaminophen reference from MetaboAnnotatoR package data to obtain candidate formulas and their fragment lists per feature. Compute the composite score S = w_mz·min(1/E,1) + w_ma·S_ma, where E is the m/z error of the candidate versus the feature, and S_ma sums the library occurrence scores of fragments matched to the pseudo-MS/MS spectrum. Down-weight by a factor of two any fragments matched only in the AIF scan (rather than the pseudo-MS/MS spectrum), because such matches are less specific. Use the default subscore weights w_mz = w_ma = 0.5 and rank candidate metabolites per feature by descending S. Cross-check the implementation against the package's compFrag, rankScore, and searchLib functions on the acetaminophen example, confirming rank scores and candidate ordering match the reference implementation.

## Related tools

- **MetaboAnnotatoR** (Provides compFrag, rankScore, and searchLib reference functions, the rCandidates and acetaminophen example data, and the fragment libraries used to compute and validate the composite score) — https://github.com/gggraca/MetaboAnnotatoR
- **R** (Implementation language in which the scoring formula is re-implemented and cross-checked against the package)

## Examples

```
library(MetaboAnnotatoR); data(rCandidates); data(acetaminophen); scores <- rankScore(rCandidates, acetaminophen)
```

## Evaluation signals

- Computed rank scores on the acetaminophen example exactly match values returned by the package's rankScore/searchLib reference implementation
- Candidate ordering per feature (descending S) matches the package's ranked candidate list
- Fragments matched only in the AIF scan contribute exactly half the weight of fragments matched in the pseudo-MS/MS spectrum
- Using default weights, the m/z-error subscore is capped at 1 (min(1/E,1) never exceeds 1)
- Downstream precision/recall for top ranks 1–5 falls within the reported 82–92% precision and 82–85% recall ranges on the human serum datasets

## Limitations

- Low recall on non-serum matrices was attributed to the small size of the fragment libraries, in particular the nonlipid library, so scores against sparse libraries may rank true candidates poorly
- Peak-shape/correlation deconvolution can yield incomplete pseudo-MS/MS spectra for co-eluting compounds with similar chemical groups, degrading S_ma matches
- Scoring was validated on QTOF AIF serum data; Orbitrap and other DIA schemes (e.g. SWATH) are untested

## Evidence

- [methods] Scoring formula combining m/z error and fragment occurrence: [evidence span withheld — anchor status: not_found]
- [methods] AIF-only fragment downweighting: [evidence span withheld — anchor status: not_found]
- [abstract] Reported top-rank performance: "Precision and recall values of 82–92% and 82–85%, respectively, were obtained for features found in the highest-rank scores (1–5)"
- [discussion] Library size limitation: "These showed modest-to-good (60–88%) precision but low recall due to the small size of the fragment libraries, in particular, the nonlipid library."
- [readme] Package installation in R: [evidence span withheld — anchor status: external_doc]
