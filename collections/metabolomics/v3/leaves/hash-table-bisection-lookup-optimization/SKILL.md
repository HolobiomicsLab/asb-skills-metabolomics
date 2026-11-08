---
name: hash-table-bisection-lookup-optimization
description: Use when implementing a large-scale MS2 spectral matching routine (e.g.,
  matching measured peaks to millions of theoretical lipid fragment m/z values within
  a ppm tolerance) and naive pairwise comparison is too slow.
when_to_use_negative:
- When input is already a processed feature/annotation table rather than raw peak
  lists requiring m/z matching.
- When the query set is small enough that direct dot-product or entropy-similarity
  computation (e.g., flash entropy search identity_search) is sufficient and a custom
  index adds no benefit.
- When annotation depends on RT-based intelligence or double-bond localization (Paternò–Büchi
  derivatization) rather than MS2 spectral matching.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: LipidIN
  role: Provides the EQ module whose hash-table and bisection data structures implement
    the fast m/z lookup and Score_matched / Score_ratio computation
  repo: github.com/LinShuhaiLAB/LipidIN
- name: flash entropy search
  role: Baseline spectral-search algorithm whose squared time complexity the hash-table/bisection
    approach of the EQ module outperforms; also a reference for index-then-search
    API design
  repo: https://github.com/YuanyueLi/FlashEntropySearch
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:b2d1df10209b137e89c6bfd8355a7c2bfe4c50621fd8219dd437115d394462e3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - flash entropy search
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# hash-table-bisection-lookup-optimization

## Summary

A data-structure optimization used in LipidIN's expeditious querying (EQ) module: theoretical fragment m/z values are indexed with hash tables and looked up with bisection, reducing spectral matching from squared to linear time complexity so Score_matched and Score_ratio can be computed at ultra-fast query speeds.

## When to use

Use when implementing a large-scale MS2 spectral matching routine (e.g., matching measured peaks to millions of theoretical lipid fragment m/z values within a ppm tolerance) and naive pairwise comparison is too slow.

## When NOT to use

- When input is already a processed feature/annotation table rather than raw peak lists requiring m/z matching.
- When the query set is small enough that direct dot-product or entropy-similarity computation (e.g., flash entropy search identity_search) is sufficient and a custom index adds no benefit.
- When annotation depends on RT-based intelligence or double-bond localization (Paternò–Büchi derivatization) rather than MS2 spectral matching.

## Inputs

- LipidIN theoretical fragment list for a candidate lipid (chain compositions + C=C double-bond positions)
- Measured MS2 spectrum (peak m/z and intensity pairs) from the deposited test spectra
- LipidIN spectral library files in Python .pkl format (e.g., pos_common.pkl, neg_common.pkl, PB_PX_H.pkl)
- Stated ppm mass tolerance parameter

## Outputs

- Per-candidate Score_matched (fraction of matched peaks)
- Per-candidate Score_ratio (ratio-based intensity similarity)
- scores_table.csv with the per-candidate score pair

## How to apply

Build a hash-table index over the theoretical fragment list (e.g., LipidIN's theoretical library of 168.6 million fragmentation entries, or the 24,631,702-spectrum general .pkl library such as pos_common.pkl / neg_common.pkl) keyed on m/z. At query time, look up each measured MS2 peak against the index using bisection within the stated ppm mass tolerance, rather than scanning all candidate fragments. For each candidate lipid, count matched peaks to compute Score_matched (fraction of matched peaks) and Score_ratio (ratio-based intensity similarity) per the decision-tree scoring of the EQ module. The rationale is that LipidIN's EQ module achieves linear time complexity via hash tables and bisection, outperforming the squared time complexity of flash entropy search, enabling speeds exceeding one hundred billion queries per second across all mass spectral libraries. Validate by running the scorer on deposited test spectra from the LipidIN spectral library (Zenodo record 21421866) and confirming score agreement with the reference implementation.

## Related tools

- **LipidIN** (Provides the EQ module whose hash-table and bisection data structures implement the fast m/z lookup and Score_matched / Score_ratio computation) — github.com/LinShuhaiLAB/LipidIN
- **flash entropy search** (Baseline spectral-search algorithm whose squared time complexity the hash-table/bisection approach of the EQ module outperforms; also a reference for index-then-search API design) — https://github.com/YuanyueLi/FlashEntropySearch

## Examples

```
from ms_entropy import FlashEntropySearch; entropy_search = FlashEntropySearch(); spectral_library = entropy_search.build_index(spectral_library); entropy_similarity = entropy_search.search(precursor_mz=query_spectrum['precursor_mz'], peaks=query_spectrum['peaks'])
```

## Evaluation signals

- Per-candidate Score_matched and Score_ratio computed by the custom scorer agree with the LipidIN reference implementation on deposited test spectra (expert_review on score agreement).
- Query throughput reaches the ultra-fast regime reported for the EQ module (linear, not squared, scaling with library size).
- All theoretical fragment m/z matches fall within the stated ppm mass tolerance (verified via bisection lookup boundaries).
- scores_table.csv contains exactly one Score_matched / Score_ratio pair per candidate lipid from the theoretical fragment list.
- Index build succeeds over the full general library (24,631,702 spectra in pos_common.pkl / neg_common.pkl) within the stated minimum 16 GB RAM.

## Limitations

- The R version of LipidIN is kept for reproducibility only and will no longer be updated; new implementations should follow the Python track.
- The squared-time-complexity flash entropy search remains the reference for entropy-based similarity; the hash-table/bisection approach targets m/z lookup speed, not entropy similarity scoring.
- Matching accuracy depends on the completeness of the theoretical fragment list (all chain compositions + C=C double-bond positions); candidates absent from the library cannot be scored.

## Evidence

- [discussion] EQ module linear-time lookup: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
- [abstract] EQ query throughput: "The expeditious querying module achieves speeds exceeding one hundred billion queries per second across all mass spectral libraries."
- [methods] EQ score pair: [evidence span withheld — anchor status: not_found]
- [readme] Flash entropy search index-then-search pattern: "# Step 1: Build the index from the library spectra
entropy_search.build_index(spectral_library)"
- [methods] LipidIN library size and format: "General (exclude PB) | Python `.pkl` (`pos_common.pkl`, `neg_common.pkl`) | 24,631,702"
