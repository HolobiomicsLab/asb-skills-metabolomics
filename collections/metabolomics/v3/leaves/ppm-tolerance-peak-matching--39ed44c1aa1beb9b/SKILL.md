---
name: ppm-tolerance-peak-matching
description: Use when a measured MS2 spectrum must be scored against theoretical fragment
  lists for candidate lipids, and per-candidate matched-peak and intensity-ratio scores
  are needed at a stated ppm tolerance.
when_to_use_negative:
- The input is already a feature table or annotation result rather than raw/processed
  MS2 peak lists needing spectral matching
- Spectral similarity should be computed by entropy similarity against a general MS/MS
  library — FlashEntropySearch (identity/hybrid/neutral-loss/open search) is the appropriate
  tool, not the lipid-specific Score_matched/Score_ratio computation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: LipidIN
  role: Provides the EQ module whose hash-table/bisection lookup performs ppm-tolerance
    peak matching and the theoretical spectral library (pos_common.pkl, neg_common.pkl)
    against which candidate lipids are scored
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: flash entropy search
  role: Baseline similarity-search algorithm; the EQ module's hash-table/bisection
    data structure is described as outperforming its squared time complexity
  repo: https://github.com/YuanyueLi/FlashEntropySearch
- name: LipidMatch
  role: Prior art for matching experimental fragment m/z values with simulated library
    m/z values from in-silico fragmentation libraries
  repo: https://github.com/GarrettLab-UF/LipidMatch
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:55c852c2fa841f7181a8313bb759fd4dfeb63cdf88bc5b927e57c86e8fd31a5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinShuhaiLAB/LipidIN
  tools:
  - LipidIN
  - LipidMatch
  - flash entropy search
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ppm-tolerance-peak-matching

## Summary

Matches measured MS2 peak m/z values against theoretical fragment m/z values from a candidate lipid within a stated ppm mass tolerance, using hash-table/bisection lookup as in the LipidIN expeditious querying (EQ) module, to compute Score_matched and Score_ratio per candidate.

## When to use

Use when a measured MS2 spectrum must be scored against theoretical fragment lists for candidate lipids, and per-candidate matched-peak and intensity-ratio scores are needed at a stated ppm tolerance.

## When NOT to use

- The input is already a feature table or annotation result rather than raw/processed MS2 peak lists needing spectral matching
- Spectral similarity should be computed by entropy similarity against a general MS/MS library — FlashEntropySearch (identity/hybrid/neutral-loss/open search) is the appropriate tool, not the lipid-specific Score_matched/Score_ratio computation

## Inputs

- Theoretical fragment list for a candidate lipid from the LipidIN spectral library (e.g. .pkl files pos_common.pkl, neg_common.pkl)
- Measured MS2 spectrum with precursor m/z and peak list (m/z, intensity pairs)
- Stated ppm mass tolerance

## Outputs

- Per-candidate Score_matched (fraction of matched peaks)
- Per-candidate Score_ratio (ratio-based intensity similarity)
- scores_table.csv with the score pair per candidate

## How to apply

Load the theoretical fragment list for each candidate lipid from the LipidIN spectral library (e.g. pos_common.pkl / neg_common.pkl for the general library) together with the measured MS2 spectrum. For each theoretical fragment m/z, look up the nearest measured peak within the stated ppm mass tolerance using a hash-table/bisection lookup, the linear-time data structure the EQ module uses to outperform squared-time flash entropy search. For each candidate compute Score_matched (fraction of matched peaks) and Score_ratio (ratio-based intensity similarity) per Equations 1-2, writing the per-candidate score pair to a scores table. These score pairs feed downstream decision trees that decide the lipid annotation.

## Related tools

- **LipidIN** (Provides the EQ module whose hash-table/bisection lookup performs ppm-tolerance peak matching and the theoretical spectral library (pos_common.pkl, neg_common.pkl) against which candidate lipids are scored) — https://github.com/LinShuhaiLAB/LipidIN
- **flash entropy search** (Baseline similarity-search algorithm; the EQ module's hash-table/bisection data structure is described as outperforming its squared time complexity) — https://github.com/YuanyueLi/FlashEntropySearch
- **LipidMatch** (Prior art for matching experimental fragment m/z values with simulated library m/z values from in-silico fragmentation libraries) — https://github.com/GarrettLab-UF/LipidMatch

## Examples

```
from ms_entropy import FlashEntropySearch; es = FlashEntropySearch(); lib = es.build_index(library_spectra); scores = es.search(precursor_mz=query['precursor_mz'], peaks=query['peaks'])
```

## Evaluation signals

- Run the scorer on the deposited test spectra and confirm Score_matched/Score_ratio agree with the LipidIN reference implementation (expert review on score agreement)
- Every emitted score pair lies in the expected [0, 1] range for both Score_matched (matched fraction) and Score_ratio
- Matched peak assignments change predictably when the ppm tolerance is tightened or loosened (more/fewer matches, monotonic in tolerance)
- scores_table.csv contains exactly one Score_matched/Score_ratio pair per candidate lipid, with no unmatched candidates silently dropped

## Limitations

- The score pair alone does not annotate a lipid — Score_matched and Score_ratio are inputs to downstream decision trees, and RT-based scores must be combined for best annotation performance
- Similarity-algorithm-based annotation can cause feature redundancy, as noted for MS-DIAL, LipidSearch, and MS entropy
- The EQ module's speed claim depends on the hash-table/bisection data structure; naive pairwise comparison over the 168.6-million-entry theoretical library will be prohibitively slow

## Evidence

- [other] EQ module is based on a non-informative prior greedy algorithm to search theoretical spectra at an ultra-fast speed: [evidence span withheld — anchor status: not_found]
- [discussion] hash-table/bisection outperforms squared-time flash entropy search: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
- [methods] theoretical library size: "168.6 million fragmentation entries (all chain compositions + C=C double-bond positions)"
- [methods] MS2 library search against theoretical library: "**Annotation** — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)"
- [readme] LipidMatch matches experimental fragment m/z with simulated library m/z: [evidence span withheld — anchor status: external_doc]
