---
name: molecular-formula-scoring-and-ranking
description: Use when IDSL.IPA per-sample peak lists must be annotated with molecular
  formulas and surviving candidate formulas (post PCS/NEME/NDCS/RCS filters) need
  a composite match score and descending rank order, per sample.
when_to_use_negative:
- Input peaks have not been processed by IDSL.IPA (no per-sample peak lists or isotopic-profile
  peak properties) — IDSL.UFA requires IDSL.IPA chromatographic information as input
- MS2-only or DDA/DIA fragmentation data is the sole evidence source — this skill
  is an MS1-only isotopic profile matching approach
- The formula space needed exceeds the supplied IPDB (e.g. exhaustive enumeration
  across C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, P is required) — use IDSL.UFAx
  instead
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: IDSL.UFA
  role: Implements the candidate-screening filters and the equation (6) composite
    score/ranking of candidate molecular formulas against IDSL.IPA peak lists using
    IPDB theoretical isotopic profiles
  repo: https://cran.r-project.org/package=IDSL.UFA
- name: IDSL.IPA
  role: Generates the per-sample peak lists and aligned peak tables (peak m/z, RT,
    isotopic-profile properties) that are the inputs to scoring and ranking
  repo: github.com/idslme/IDSL.IPA
- name: R
  role: Runtime environment in which IDSL.UFA and IDSL.IPA are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c00563
    title: IDSL.UFA Assigns High-Confidence Molecular Formula Annotations for Untargeted
      LC/HRMS Data Sets in Metabolomics and Exposomics
  source_content_hash: sha256:b28590b83f25c912e9e7bcace376d6c9971375de8907bd2ee07682e660665ad0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IDSL.IPA
  - IDSL.UFA
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# molecular-formula-scoring-and-ranking

## Summary

Scores and ranks candidate molecular formulas for an LC/HRMS peak by combining profile cosine similarity (PCS), retention-of-candidates score (RCS), normalized Euclidean mass error (NEME), and the experimental-to-theoretical R13C ratio into the composite score of equation (6) of IDSL.UFA, after candidate-screening filters have been applied.

## When to use

Use when IDSL.IPA per-sample peak lists must be annotated with molecular formulas and surviving candidate formulas (post PCS/NEME/NDCS/RCS filters) need a composite match score and descending rank order, per sample.

## When NOT to use

- Input peaks have not been processed by IDSL.IPA (no per-sample peak lists or isotopic-profile peak properties) — IDSL.UFA requires IDSL.IPA chromatographic information as input
- MS2-only or DDA/DIA fragmentation data is the sole evidence source — this skill is an MS1-only isotopic profile matching approach
- The formula space needed exceeds the supplied IPDB (e.g. exhaustive enumeration across C, B, Br, Cl, K, S, Si, N, H, As, F, I, Na, O, P is required) — use IDSL.UFAx instead

## Inputs

- IDSL.IPA per-sample peak list CSV (peak m/z, RT, isotopic profile properties such as nIsoPair and peak area)
- IPDB isotopic profile database (.Rdata) generated for the formula source (e.g. RefMetDB positive/negative mode IPDB)
- IDSL.IPA peak_alignment directory (aligned peak table) for cross-sample rank summarization
- UFA parameter spreadsheet (XLSX) with scoring/ranking parameters

## Outputs

- Per-sample annotated peak list with rank, composite score (equation 6), molecular formula and IPA peak properties for each surviving candidate
- Aligned annotated peak table with detection frequency and median rank summaries per peak

## How to apply

Load the IDSL.IPA per-sample peak list csv and the IPDB (isotopic profile database, .Rdata) into R via IDSL.UFA. First screen candidates by the article's filters on peak properties — profile cosine similarity (PCS), normalized Euclidean mass error (NEME), number of isotope-pair correlation (NDCS) and retention-of-candidates score (RCS) criteria — so only eligible formulas survive. For each surviving formula, compare its theoretical isotopic profile computed from the IPDB with the observed MS1 isotopic profile of the peak, then combine the per-hit metrics into the composite score defined by equation (6), with parameters and default value 1 applied to each component (PCS, RCS, NEME, and the experimental-to-theoretical R13C ratio). Rank candidates in descending score order and assemble the annotation table with rank, score, molecular formula and IPA-derived peak properties such as nIsoPair, RCS and peak area. In the aligned table across samples, summarize annotations by detection frequency and median rank; the paper reports hits as high-confidence when the median rank is ≤ 5 (top rank ≤ 30 and number of hits ≤ 30 for unique formula reporting in aligned tables).

## Related tools

- **IDSL.UFA** (Implements the candidate-screening filters and the equation (6) composite score/ranking of candidate molecular formulas against IDSL.IPA peak lists using IPDB theoretical isotopic profiles) — https://cran.r-project.org/package=IDSL.UFA
- **IDSL.IPA** (Generates the per-sample peak lists and aligned peak tables (peak m/z, RT, isotopic-profile properties) that are the inputs to scoring and ranking) — github.com/idslme/IDSL.IPA
- **R** (Runtime environment in which IDSL.UFA and IDSL.IPA are executed)

## Examples

```
library(IDSL.UFA); UFA_workflow("Address of the UFA parameter spreadsheet")
```

## Evaluation signals

- Every peak in the annotated output csv carries rank, score and molecular-formula columns populated for all surviving candidates
- Candidate formulas excluded by PCS/NEME/NDCS/RCS screening criteria do not appear in the scored list
- Reported high-confidence annotations satisfy the median rank ≤ 5 criterion (top rank ≤ 30 and number of hits ≤ 30 for unique formulas in aligned tables)
- The experimental-to-theoretical R13C ratio, PCS, NEME and RCS components are all present with the parameter/default value of 1 weighting applied in the equation (6) score
- Detection-frequency and median-rank summaries in the aligned annotated peak table are consistent across samples for the same peak

## Limitations

- Database-dependent screening means compounds absent from the supplied formula source (e.g. poorly represented exposure compounds in biochemical databases) may be missed
- Performance depends on IPDB coverage and isotopic-profile accuracy; the paper reports 54.31–85.51% of true positives as top hit and 90.58–100% within top five hits, not 100% top-hit recovery
- Scoring is MS1-only; formula assignments are not disambiguated by tandem mass spectrometry unless MS2 evidence is supplied separately

## Evidence

- [methods] IDSL.UFA computes a matching score for each candidate filtered formula via equation (6), which combines Profile cosine similarity (PCS), retention-of-candidates score (RCS), normalized Euclidean mass error (NEME), and the experimental-to-theoretical R13C ratio, with parameters and default values of 1 applied to each; the score determines the rank of candidate molecular formulas.: [evidence span withheld — anchor status: not_found]
- [readme] The README describes generating comprehensive in-silico theoretical libraries known as IPDB using natural isotopic distribution profiles.: [evidence span withheld — anchor status: external_doc]
- [readme] The README describes IDSL.UFA ranking candidate molecular formulas for a peak as a key feature.: [evidence span withheld — anchor status: external_doc]
- [readme] The README workflow states that mass spectrometry data should be processed using the IDSL.IPA workflow to acquire chromatographic information of the peaks (m/z-RT) before UFA annotation.: [evidence span withheld — anchor status: external_doc]
- [readme] The README quick-batch example names the IPDB from the RefMetDB folder according to IDSL.IPA results as an input parameter.: [evidence span withheld — anchor status: external_doc]
