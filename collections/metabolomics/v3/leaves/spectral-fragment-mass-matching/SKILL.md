---
name: spectral-fragment-mass-matching
description: Use when a measured MS2 spectrum must be scored against theoretical lipid
  fragment lists — e.g., to populate Score_matched and Score_ratio for annotation
  decision trees at a stated ppm mass tolerance.
when_to_use_negative:
- Input is already a feature table or annotated lipid list rather than raw MS2 peak
  lists needing fragment matching
- The task is general small-molecule MS/MS similarity search against public spectral
  libraries — flash entropy search / ms-entropy is the appropriate tool, and LipidIN's
  EQ module outperforms it only in the lipid-library context described here
- Waters-file-derived spectra or file formats LipidMatch explicitly does not support
  and that LipidIN's documented inputs (Thermo .raw, SCIEX .wiff, Bruker/Agilent .d,
  .mzML) do not cover
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: LipidIN
  role: Provides the EQ module (hash-table/bisection fragment matching, non-informative
    prior greedy search) and the theoretical spectral libraries from which candidate
    fragment lists are loaded
  repo: https://github.com/LinShuhaiLAB/LipidIN
- name: flash entropy search
  role: Baseline spectral-similarity search algorithm against which the EQ module's
    linear-time hash-table/bisection lookup is benchmarked
  repo: https://github.com/YuanyueLi/FlashEntropySearch
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:905aba880f9fe201c24b0e89be66f7548296708adc856f9f10e7736073e5bb27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinShuhaiLAB/LipidIN
  tools:
  - LipidIN
  - flash entropy search
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spectral-fragment-mass-matching

## Summary

Match measured MS2 fragment m/z values against theoretical lipid fragment libraries within a stated ppm tolerance, computing Score_matched (fraction of matched peaks) and Score_ratio (ratio-based intensity similarity) per LipidIN's EQ module for candidate lipid scoring.

## When to use

Use when a measured MS2 spectrum must be scored against theoretical lipid fragment lists — e.g., to populate Score_matched and Score_ratio for annotation decision trees at a stated ppm mass tolerance.

## When NOT to use

- Input is already a feature table or annotated lipid list rather than raw MS2 peak lists needing fragment matching
- The task is general small-molecule MS/MS similarity search against public spectral libraries — flash entropy search / ms-entropy is the appropriate tool, and LipidIN's EQ module outperforms it only in the lipid-library context described here
- Waters-file-derived spectra or file formats LipidMatch explicitly does not support and that LipidIN's documented inputs (Thermo .raw, SCIEX .wiff, Bruker/Agilent .d, .mzML) do not cover

## Inputs

- Measured MS2 spectrum (peak list of m/z and intensity) from deposited test spectra
- Theoretical fragment list for a candidate lipid from the LipidIN theoretical spectral library (.pkl files such as pos_common.pkl, neg_common.pkl, or .rda pos_ALL.rda/neg_ALL.rda)
- Stated ppm mass tolerance for fragment matching

## Outputs

- Per-candidate Score_matched (fraction of matched peaks)
- Per-candidate Score_ratio (ratio-based intensity similarity)
- scores_table.csv with the score pair for each candidate lipid

## How to apply

Load the theoretical fragment list for each candidate lipid from the LipidIN spectral library (e.g., pos_common.pkl / neg_common.pkl from Zenodo record 21421866) and the measured MS2 spectrum from the deposited test spectra. Match theoretical fragment m/z values to measured peaks within the stated ppm mass tolerance using the hash-table/bisection lookup strategy of the EQ module, which LipidIN reports achieves linear time complexity (outperforming squared time complexity in flash entropy search) at speeds exceeding one hundred billion queries per second. For each candidate compute Score_matched as the fraction of matched peaks and Score_ratio as the ratio-based intensity similarity exactly per the module's Equations 1-2. Write the per-candidate score pair to a table (e.g., scores_table.csv) for downstream decision-tree annotation, and validate by confirming score agreement with the LipidIN reference implementation.

## Related tools

- **LipidIN** (Provides the EQ module (hash-table/bisection fragment matching, non-informative prior greedy search) and the theoretical spectral libraries from which candidate fragment lists are loaded) — https://github.com/LinShuhaiLAB/LipidIN
- **flash entropy search** (Baseline spectral-similarity search algorithm against which the EQ module's linear-time hash-table/bisection lookup is benchmarked) — https://github.com/YuanyueLi/FlashEntropySearch

## Evaluation signals

- Run the scorer on the deposited test spectra and confirm Score_matched/Score_ratio outputs agree with the LipidIN reference implementation (expert_review on score agreement)
- Score_matched must lie in [0, 1], being the fraction of matched peaks
- Matched fragment assignments are all within the stated ppm mass tolerance of the theoretical m/z values
- scores_table.csv contains a score pair (Score_matched, Score_ratio) for every candidate lipid evaluated
- Lookup throughput remains consistent with the EQ module's hash-table/bisection design (linear rather than squared time complexity in library size)

## Limitations

- Scores from similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy can cause feature redundancy; EQ-based scoring is positioned to avoid this but the score pair alone is intended to feed decision trees, not replace RT-based checks
- The EQ module was developed and validated for the LipidIN lipid theoretical libraries; applying it to non-lipid spectral libraries is outside the reported scope
- Combining MS/MS-based scores with RT-based scores is needed for the strongest annotation performance; fragment matching alone gives only part of the annotation evidence

## Evidence

- [methods] EQ module greedy/ppm matching: [evidence span withheld — anchor status: not_found]
- [discussion] hash tables and bisection, linear time: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
- [abstract] query throughput: "The expeditious querying module achieves speeds exceeding one hundred billion queries per second across all mass spectral libraries."
- [discussion] feature redundancy caveat: "Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy."
- [methods] library files and sizes: "General (exclude PB) | Python `.pkl` (`pos_common.pkl`, `neg_common.pkl`) | 24,631,702"
- [readme] readme annotation definition: "**Annotation** — MS2 library search against the theoretical library (chain compositions + C=C double-bond positions)."
