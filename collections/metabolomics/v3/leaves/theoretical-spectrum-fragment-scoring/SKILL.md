---
name: theoretical-spectrum-fragment-scoring
description: Use when a measured MS2 spectrum must be scored against candidate lipid
  theoretical fragment spectra (m/z matching within ppm tolerance) to produce Score_matched
  and Score_ratio pairs for lipid annotation decision trees.
when_to_use_negative:
- Input is already an annotated feature table or quantification matrix — this skill
  operates on raw MS2 peak lists against theoretical fragments
- Query is general small-molecule MS/MS similarity search against an experimental
  spectral library rather than lipid theoretical-fragment scoring — flash entropy
  search / MS entropy similarity is the appropriate alternative
- No theoretical fragment list exists for the candidate lipid class (e.g. subclasses
  outside the 91/108 subclass×adduct combinations of the LipidIN general library)
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidIN
  role: Provides the EQ module (hash-table/bisection lookup, non-informative prior
    greedy algorithm) used to compute Score_matched and Score_ratio, plus the theoretical
    fragment library being searched
  repo: github.com/LinShuhaiLAB/LipidIN
- name: flash entropy search
  role: Alternative/baseline spectral similarity algorithm compared against the EQ
    module in the article; its squared time complexity is outperformed by EQ's linear-time
    hash-table approach
  repo: https://github.com/YuanyueLi/MSEntropy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:647870e478594befcdcebcf59b3fe9d1e6f40fdeddc726e3b950c3a6a4af0a4d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - flash entropy search
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# theoretical-spectrum-fragment-scoring

## Summary

Scores candidate lipid identifications by matching measured MS2 peak m/z values against theoretical fragment lists within a stated ppm tolerance and computing Score_matched (fraction of matched peaks) and Score_ratio (ratio-based intensity similarity), as implemented in LipidIN's expeditious querying (EQ) module for downstream decision trees.

## When to use

Use when a measured MS2 spectrum must be scored against candidate lipid theoretical fragment spectra (m/z matching within ppm tolerance) to produce Score_matched and Score_ratio pairs for lipid annotation decision trees.

## When NOT to use

- Input is already an annotated feature table or quantification matrix — this skill operates on raw MS2 peak lists against theoretical fragments
- Query is general small-molecule MS/MS similarity search against an experimental spectral library rather than lipid theoretical-fragment scoring — flash entropy search / MS entropy similarity is the appropriate alternative
- No theoretical fragment list exists for the candidate lipid class (e.g. subclasses outside the 91/108 subclass×adduct combinations of the LipidIN general library)

## Inputs

- Measured MS2 peak list (m/z, intensity pairs) for the query feature
- Theoretical fragment lists for candidate lipids from the LipidIN theoretical spectral library (e.g. pos_common.pkl / neg_common.pkl or pos_ALL.rda / neg_ALL.rda)
- Stated ppm mass tolerance parameter for fragment matching

## Outputs

- Per-candidate Score_matched and Score_ratio pairs (scores_table.csv)
- Ranked candidate lipid annotations feeding downstream decision trees

## How to apply

Load the theoretical fragment list for each candidate lipid from the LipidIN theoretical library (168.6 million fragmentation entries covering all chain compositions and C=C double-bond positions, distributed as .pkl files such as pos_common.pkl/neg_common.pkl or R .rda files) together with the measured MS2 peak list from the deposited test spectra. Match each theoretical fragment m/z to measured peaks within the stated ppm mass tolerance using the hash-table/bisection lookup of the EQ module, which achieves linear time complexity and was chosen over the squared time complexity of flash entropy search. For each candidate compute Score_matched (fraction of matched peaks) and Score_ratio (ratio-based intensity similarity) exactly per Equations 1-2, then write the per-candidate score pair to scores_table.csv. Validate by running the scorer on the deposited test spectra and confirming the outputs agree with the LipidIN reference implementation (expert_review on score agreement).

## Related tools

- **LipidIN** (Provides the EQ module (hash-table/bisection lookup, non-informative prior greedy algorithm) used to compute Score_matched and Score_ratio, plus the theoretical fragment library being searched) — github.com/LinShuhaiLAB/LipidIN
- **flash entropy search** (Alternative/baseline spectral similarity algorithm compared against the EQ module in the article; its squared time complexity is outperformed by EQ's linear-time hash-table approach) — https://github.com/YuanyueLi/MSEntropy

## Examples

```
from ms_entropy import FlashEntropySearch; es = FlashEntropySearch(); es.build_index(spectral_library); es.search(precursor_mz=q['precursor_mz'], peaks=q['peaks'])
```

## Evaluation signals

- Score pairs reproduce the LipidIN reference implementation on the deposited test spectra (expert_review on score agreement)
- Every matched peak lies within the stated ppm mass tolerance of a theoretical fragment m/z
- Score_matched is a fraction in [0, 1] by construction (fraction of matched peaks)
- Matching completes with hash-table/bisection linear-time lookup; throughput is consistent with the EQ module's billion-query-per-second scale
- Rerunning the scorer on identical inputs yields identical per-candidate score pairs (deterministic greedy, non-informative prior)

## Limitations

- The EQ module uses a non-informative prior greedy algorithm; scores are intended for downstream decision trees rather than as standalone confidence values
- Scoring requires candidate theoretical fragment lists; it is specific to lipid annotation and cannot be applied without the LipidIN theoretical library or equivalent in-silico fragments
- The article notes flash entropy search has squared time complexity, so substituting generic entropy similarity for the EQ lookup changes both the scoring semantics and the scaling

## Evidence

- [discussion] EQ hash-table/bisection design and speed advantage over flash entropy search: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
- [abstract] EQ module throughput: "The expeditious querying module achieves speeds exceeding one hundred billion queries per second across all mass spectral libraries."
- [readme] Theoretical library scope for fragment matching: "Theoretical library size | 168.6 million fragmentation entries (all chain compositions + C=C double-bond positions)"
- [methods] Annotation step this scoring feeds: "**Annotation** — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)"
