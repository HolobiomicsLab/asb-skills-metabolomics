---
name: overlap-statistics-aggregation
description: Use when validating a new HRMS peak-picking workflow against established
  open-source tools (MSDIAL, XCMS) by matching m/z–RT feature pairs across pipelines
  on samples of known composition and reporting overlap fractions.
when_to_use_negative:
- Input is already a single annotated or merged feature table — this skill compares
  raw independently extracted m/z–RT lists and would double-count matched features
- The goal is compound identification (annotation, MS/MS fragment matching) rather
  than benchmarking feature-extraction agreement
- Samples are of unknown composition where a disagreement between tools cannot be
  attributed to either pipeline
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Mass-Suite (MSS)
  role: Produces the candidate feature lists whose overlap with reference tools is
    being quantified
  repo: https://github.com/XiminHu/mass-suite
- name: MSDIAL
  role: Open-source reference platform generating one of the comparison feature lists
    from the same .mzML files
- name: XCMS
  role: Open-source reference platform generating the second comparison feature list
    from the same .mzML files
- name: Python / Pandas
  role: Holds feature lists in DataFrames, performs m/z–RT proximity matching, fraction
    computation, and aggregation to CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:2e6bf7b61d2af6154c518b199bb8df56d0601ab51944e4236eee7f9b5d5eb8dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - MSDIAL
  - Mass-Suite (MSS)
  - Pandas
  - Python
  - Python / Pandas
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# overlap-statistics-aggregation

## Summary

Quantify the reproducibility of HRMS feature extraction by computing per-sample overlap fractions between Mass-Suite (MSS) m/z–RT feature lists and reference lists from MSDIAL or XCMS, then aggregating them into mean overlap ± standard deviation for benchmarking.

## When to use

Use when validating a new HRMS peak-picking workflow against established open-source tools (MSDIAL, XCMS) by matching m/z–RT feature pairs across pipelines on samples of known composition and reporting overlap fractions.

## When NOT to use

- Input is already a single annotated or merged feature table — this skill compares raw independently extracted m/z–RT lists and would double-count matched features
- The goal is compound identification (annotation, MS/MS fragment matching) rather than benchmarking feature-extraction agreement
- Samples are of unknown composition where a disagreement between tools cannot be attributed to either pipeline

## Inputs

- MSS-extracted feature lists (m/z–RT pairs with peak areas) from mssmain.peak_pick on .mzML files for ENTACT samples #505, #506, and #508
- Reference feature lists exported from MSDIAL run on the same .mzML files
- Reference feature lists exported from XCMS run on the same .mzML files
- MSS-reported per-sample feature counts (overlap denominators)

## Outputs

- Per-sample overlap fractions for each comparison platform
- Aggregated mean overlap ± standard deviation per platform (overlap_fractions.csv)
- Counts of matched (shared) features between MSS and each reference tool

## How to apply

Load the MSS-extracted feature lists (m/z–RT pairs with peak areas) for ENTACT mixture samples #505, #506, and #508 into pandas DataFrames, and load the corresponding MSDIAL/XCMS feature lists run on the same .mzML files with peak-picking parameters matched as closely as possible. For each sample–platform pair, match features by proximity in the m/z–RT space, consistent with the Euclidean-distance alignment MSS uses, and count shared features. Compute the overlap fraction per sample as shared features divided by the MSS-reported feature count, then aggregate across the three samples as mean overlap ± standard deviation, writing per-sample and aggregated values to overlap_fractions.csv. The published benchmark is the validation criterion: ~52 ± 5% overlap for MSDIAL and ~52 ± 6% for XCMS against MSS's 99.5% extraction accuracy (398 known ENTACT peaks, 2 unmatched).

## Related tools

- **Mass-Suite (MSS)** (Produces the candidate feature lists whose overlap with reference tools is being quantified) — https://github.com/XiminHu/mass-suite
- **MSDIAL** (Open-source reference platform generating one of the comparison feature lists from the same .mzML files)
- **XCMS** (Open-source reference platform generating the second comparison feature list from the same .mzML files)
- **Python / Pandas** (Holds feature lists in DataFrames, performs m/z–RT proximity matching, fraction computation, and aggregation to CSV)

## Examples

```
import pandas as pd; mss = pd.read_csv('mss_505.csv'); xcms = pd.read_csv('xcms_505.csv'); frac = len(set(map(tuple, mss[['mz','rt']].values)) & set(map(tuple, xcms[['mz','rt']].values))) / len(mss)
```

## Evaluation signals

- Aggregated mean overlap should be ~52 ± 5% for MSDIAL and ~52 ± 6% for XCMS, matching the reported benchmark
- Every per-sample overlap fraction is bounded in [0, 1], with the denominator equal to the MSS-reported feature count for that sample
- Per-sample and aggregated fractions are both present in overlap_fractions.csv, broken out separately for MSDIAL and XCMS
- Cross-check that MSS itself captured 99.5% of the 398 known ENTACT peaks (2 unmatched), so disagreement is attributable to tool differences rather than missed MSS features

## Limitations

- Overlap is asymmetric — fractions are normalized by the MSS feature count, so the statistic measures MSS coverage of other tools' features, not symmetric Jaccard similarity
- Peak-picking parameters across platforms could only be 'matched as closely as possible', so residual overlap gaps may reflect parameterization, not algorithmic differences
- Feature grouping (merging isotopes, adducts, and in-source fragments) is not yet implemented in MSS, which can inflate or deflate feature counts relative to grouped outputs of other tools

## Evidence

- [methods] Benchmark of feature overlap across pipelines: "on average 52 ± 5% and 52 ± 6%, for MSDIAL and XCMS respectively), validating MSS performance"
- [methods] Ground-truth completeness of MSS extraction: "MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures"
- [readme] Rationale for m/z–RT proximity matching: "Data from different experiments will be aligned together according to user defined error range."
