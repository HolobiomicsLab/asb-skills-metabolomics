---
name: metabolite-structural-class-pair-categorization
description: Use when a SIMILE/cosine spectral-alignment results table must be labeled
  by metabolite structural class (flavonoid vs. indole, etc.) and uniquely-SIMILE
  pairs need MCS-category annotation and mass-delta interpretation.
when_to_use_negative:
- Compound pairs lack structural annotations or computable structures, so structural-class
  labels and MCS categories cannot be assigned.
- The task is computing the spectral-alignment significance itself (similarity_matrix/z_test
  pipeline) rather than interpreting and categorizing already-computed pair calls.
- Only intermediate similarity scores without statistical significance are available
  — the paper notes such scores are 'to some degree uninterpretable' for prioritizing
  metabolites.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3335
tools:
- name: SIMILE
  role: Generates the per-pair significance calls, mass deltas and scores (via sml.matching_ions_report)
    that are then categorized by structural class and MCS category
  repo: https://mybinder.org/v2/gh/biorack/simile/HEAD
- name: GNPS
  role: Provides the cosine-based spectral similarity calls that serve as the comparison
    baseline when labeling pairs as SIMILE-unique vs. shared
- name: pandas
  role: Assembles and saves the pair-categorization table (simile_vs_cosine_pairs.csv)
    with structural-class and MCS annotations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:06742c7a2bfa19dad5acdbd0da5d3f90bf235acf6c2813d955a666227ac78c7f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - SIMILE
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3335
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# metabolite-structural-class-pair-categorization

## Summary

Categorizes interspectral compound pairs from SIMILE vs. GNPS-cosine spectral-alignment comparisons by structural class (e.g., 20 flavonoids and 20 indoles from the Berkeley Lab GNPS reference library) and assigns MCS categories and mass-delta interpretations (e.g., 30.983 Da as [+O2 −H]) to pairs found uniquely by SIMILE.

## When to use

Use when a SIMILE/cosine spectral-alignment results table must be labeled by metabolite structural class (flavonoid vs. indole, etc.) and uniquely-SIMILE pairs need MCS-category annotation and mass-delta interpretation.

## When NOT to use

- Compound pairs lack structural annotations or computable structures, so structural-class labels and MCS categories cannot be assigned.
- The task is computing the spectral-alignment significance itself (similarity_matrix/z_test pipeline) rather than interpreting and categorizing already-computed pair calls.
- Only intermediate similarity scores without statistical significance are available — the paper notes such scores are 'to some degree uninterpretable' for prioritizing metabolites.

## Inputs

- SIMILE matching-ions report table from sml.matching_ions_report(S, M, C, mzs, pmzs) with mass deltas and scores per pair
- Per-pair SIMILE significance calls (z_test p-values) and GNPS-cosine similarity calls
- Curated compound list with structural classes (20 flavonoids and 20 indoles from the Berkeley Lab public GNPS reference library) and structures for MCS computation

## Outputs

- simile_vs_cosine_pairs.csv listing every compound pair's SIMILE significance call, GNPS-cosine call, mass deltas and scores
- Structural-class labels (flavonoid/indole) for all 780 pairs (40 choose 2)
- MCS categories for compound pairs found uniquely by SIMILE

## How to apply

Compile the pair-level output of sml.matching_ions_report(S, M, C, mzs, pmzs) into a table (simile_vs_cosine_pairs.csv) listing each pair's SIMILE significance call, GNPS-cosine call, mass deltas and scores. Label each of the 40 compounds (20 flavonoids, 20 indoles) with its structural class so every pair can be categorized as within-flavonoid, within-indole, or cross-class, and verify all 780 pairs (40 choose 2) receive both calls. For pairs flagged significant by SIMILE but not by cosine, compute maximum common substructure (MCS) categories to characterize the multiple structural modifications that cosine-based scoring misses. Interpret the reported mass deltas chemically — e.g., the predominant difference of 30.983 likely corresponds to [+O2 −H], consistent with addition of two oxygens — to connect alignment calls to modification patterns.

## Related tools

- **SIMILE** (Generates the per-pair significance calls, mass deltas and scores (via sml.matching_ions_report) that are then categorized by structural class and MCS category) — https://mybinder.org/v2/gh/biorack/simile/HEAD
- **GNPS** (Provides the cosine-based spectral similarity calls that serve as the comparison baseline when labeling pairs as SIMILE-unique vs. shared)
- **pandas** (Assembles and saves the pair-categorization table (simile_vs_cosine_pairs.csv) with structural-class and MCS annotations)

## Examples

```
df = sml.matching_ions_report(S, M, C, mzs, pmzs); df.to_csv('simile_vs_cosine_pairs.csv')
```

## Evaluation signals

- The compiled table reports both a SIMILE call and a GNPS-cosine call for all 780 pairs (40 choose 2) of the 20-flavonoid/20-indole dataset.
- The set of SIMILE-significant pairs is a superset-different enrichment relative to cosine hits, consistent with SIMILE yielding 'different and typically more associations than cosine-based scoring algorithms'.
- Every pair found uniquely by SIMILE carries an assigned MCS category reflecting multiple structural modifications (as in Supplementary Fig. SI5 and Supplementary Dataset 1, with 97 flavonoid pairs).
- Reported mass deltas are chemically interpretable, e.g. the predominant difference of 30.983 corresponding to [+O2 −H].
- Structural-class labels partition pairs into within-class and cross-class categories consistent with the curated 20-flavonoid/20-indole input.

## Limitations

- Significant SIMILE alignment of fragmentation spectra does not necessitate compound structure similarity, though significant alignments often correspond to moderate-to-high structural similarity.
- Likely there will be classes of molecules that are more appropriate for cosine-based scoring than SIMILE-based scoring; unknown which classes until categorized.
- Parameters used to compare alignment algorithms were not optimized, and optimization entails unquantified trade-offs such as improving specificity vs. number of hits.
- SIMILE is currently limited to analysis of protonation/deprotonation, constraining the adduct context of the categorized pairs.

## Evidence

- [abstract] SIMILE, applied to two molecular networking datasets, identified spectral alignments between small molecules with multiple differences that were not found by cosine-based similarity scores: [evidence span withheld — anchor status: not_found]
- [discussion] Significant SIMILE alignment does not necessitate compound structure similarity: "It is important to note that significant SIMILE alignment of fragmentation spectra does not necessitate compound structure similarity."
- [discussion] the predominant m/z difference in Fig SI6a of 30.983 likely corresponds to [+O2 −H]: "the predominant m/z difference in Fig SI6a of 30.983 likely corresponds to [+O2 −H]"
- [readme] Report back mass deltas and scores for simile comparison: "# Report back mass deltas and scores for simile comparison
df = sml.matching_ions_report(S, M, C, mzs, pmzs)"
