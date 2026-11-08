---
name: hrms-feature-list-overlap-analysis
description: Use when you have feature lists (m/z–RT pairs with peak areas) from one
  HRMS peak-picking tool and equivalent lists from comparison tools run on identical
  .mzML files of known-composition standards, and need a cross-platform overlap fraction
  to validate extraction performance.
when_to_use_negative:
- Input is a single platform's feature table with no reference-tool output to compare
  against — an overlap fraction cannot be computed
- Samples are not of known composition (e.g., environmental samples without ENTACT-style
  standards) — overlap alone cannot validate extraction accuracy; use the known-peaks
  accuracy check (99.5% of 398 peaks) instead
- Feature lists come from raw files processed with non-matched peak-picking parameters
  across platforms — differences would reflect parameter choices rather than extraction
  performance
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Mass-Suite (MSS)
  role: Primary feature-extraction tool whose output feature lists are benchmarked
    against reference tools; uses mssmain.peak_pick and Euclidean-distance alignment
  repo: https://github.com/XiminHu/mass-suite
- name: MSDIAL
  role: Reference open-source HRMS tool producing comparison feature lists on the
    same .mzML files
- name: XCMS
  role: Reference open-source HRMS tool producing comparison feature lists on the
    same .mzML files
- name: Pandas
  role: Loads and manipulates the feature-list DataFrames for matching and aggregation
- name: pymzML
  role: Underlying mzML parser used by MSS to import and parse raw data (mssmain.get_scans)
  repo: https://github.com/pymzml/pymzML
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:c735e54e346a733201bb200763916460a65b542209baa8637dfbc5d6d51d715b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - MSDIAL
  - Mass-Suite (MSS)
  - Pandas
  - Python
  - XCMS
  - pymzML
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# hrms-feature-list-overlap-analysis

## Summary

Benchmarks an HRMS feature-extraction pipeline (e.g., Mass-Suite/MSS) by quantifying how many of its extracted m/z–RT features overlap with features extracted by reference open-source tools such as MSDIAL and XCMS on the same mzML files, replicating the ~52% overlap reported for MSS on EPA ENTACT validation mixtures.

## When to use

Use when you have feature lists (m/z–RT pairs with peak areas) from one HRMS peak-picking tool and equivalent lists from comparison tools run on identical .mzML files of known-composition standards, and need a cross-platform overlap fraction to validate extraction performance.

## When NOT to use

- Input is a single platform's feature table with no reference-tool output to compare against — an overlap fraction cannot be computed
- Samples are not of known composition (e.g., environmental samples without ENTACT-style standards) — overlap alone cannot validate extraction accuracy; use the known-peaks accuracy check (99.5% of 398 peaks) instead
- Feature lists come from raw files processed with non-matched peak-picking parameters across platforms — differences would reflect parameter choices rather than extraction performance

## Inputs

- MSS-extracted feature lists (m/z–RT pairs with peak areas) as CSV/DataFrames for ENTACT samples #505, #506, #508
- MSDIAL-exported feature lists for the same .mzML files
- XCMS-exported feature lists for the same .mzML files
- .mzML raw files processed identically by all three platforms

## Outputs

- overlap_fractions.csv with per-sample overlap fractions for MSDIAL and XCMS
- Aggregated mean overlap ± standard deviation per comparison platform (expected ~52 ± 5% MSDIAL, ~52 ± 6% XCMS)

## How to apply

Load the MSS-extracted feature lists (m/z–RT pairs with peak areas) for ENTACT mixture samples #505, #506, and #508 into pandas DataFrames, and load the corresponding MSDIAL and XCMS feature lists exported from the same .mzML files with peak-picking parameters matched as closely as possible across platforms. For each sample and each comparison platform, match features by proximity in m/z–RT space, consistent with the Euclidean-distance alignment used in MSS, counting shared features. Compute the per-sample overlap fraction as shared features divided by the MSS-reported feature count, then aggregate across the three samples to report mean overlap ± standard deviation for MSDIAL and XCMS separately, and write per-sample and aggregated values to overlap_fractions.csv.

## Related tools

- **Mass-Suite (MSS)** (Primary feature-extraction tool whose output feature lists are benchmarked against reference tools; uses mssmain.peak_pick and Euclidean-distance alignment) — https://github.com/XiminHu/mass-suite
- **MSDIAL** (Reference open-source HRMS tool producing comparison feature lists on the same .mzML files)
- **XCMS** (Reference open-source HRMS tool producing comparison feature lists on the same .mzML files)
- **Pandas** (Loads and manipulates the feature-list DataFrames for matching and aggregation)
- **pymzML** (Underlying mzML parser used by MSS to import and parse raw data (mssmain.get_scans)) — https://github.com/pymzml/pymzML

## Examples

```
import pandas as pd; mss_feats = pd.read_csv('mss_505.csv'); msd_feats = pd.read_csv('msdial_505.csv'); overlap = len(set(map(tuple, mss_feats[['mz','rt']].values)) & set(map(tuple, msd_feats[['mz','rt']].values))) / len(mss_feats)
```

## Evaluation signals

- Aggregated mean overlap ≈ 52 ± 5% for MSDIAL and ≈ 52 ± 6% for XCMS, matching article-reported values
- Per-sample overlap fractions written for each of the three samples (#505, #506, #508) and each platform in overlap_fractions.csv
- MSS-reported feature count is the denominator for every overlap fraction (shared features / MSS count)
- Feature matching uses proximity in m/z–RT space consistent with MSS Euclidean-distance alignment
- MSS baseline sanity check: MSS extracts 99.5% of the 398 known peaks in the ENTACT mixtures (2 unmatched)

## Limitations

- Peak-picking parameters must be matched as closely as possible across platforms; residual parameter differences can bias overlap fractions
- Overlap fraction is relative to the MSS feature count only, so it measures consistency with reference tools, not absolute correctness
- The MSS package is actively maintained and its version is not pinned in the article, and feature grouping (merging isotopes, adducts, in-source fragments) is not yet implemented, which can inflate MSS feature counts relative to grouped-feature tools

## Evidence

- [methods] On the validation dataset, the fraction of MSS-extracted features overlapping other open-source tools averaged 52 ± 5% for MSDIAL and 52 ± 6% for XCMS, validating MSS performance.: "on average 52 ± 5% and 52 ± 6%, for MSDIAL and XCMS respectively), validating MSS performance"
- [methods] archived samples (mixed chemical standards) from the EPA ENTACT study [73] (Additional file 1: Text S2; sample numbers #505, #506 and #508; 398 MS-amenable chemicals in total): "archived samples (mixed chemical standards) from the EPA ENTACT study [73] (Additional file 1: Text S2; sample numbers #505, #506 and #508; 398 MS-amenable chemicals in total)"
- [methods] MSS extracted 99.5% of peaks (2 peaks out of 398 didn't match) known to be present in all three mixtures [73].: [evidence span withheld — anchor status: not_found]
- [abstract] MSS reliability was validated with mixed chemical standards of known composition, with 99.5% feature extraction accuracy and ~ 52% overlap of extracted features relative to other open-source software tools: "and ~ 52% overlap of extracted features relative to other open-source software tools"
- [readme] README update note confirming the validation comparison against MSDIAL:: "2020/09/01: Feature extraction pipeline validated comparing with msdial pipeline output"
