---
name: cross-platform-feature-matching
description: Use when validating a new/in-house HRMS feature-extraction workflow by
  measuring the fraction of its m/z–RT features recovered by reference tools like
  MSDIAL or XCMS on identical mzML files of samples with known composition.
when_to_use_negative:
- Input is still raw .mzML data rather than exported per-tool feature lists — first
  run each platform's feature extraction and export its feature table before matching
- The goal is absolute extraction accuracy (recovery of the 398 known ENTACT peaks,
  99.5% accuracy) rather than cross-tool overlap — that is a separate check against
  ground-truth composition
- Only one platform's output is available — overlap fractions require MSS and at least
  one reference tool (MSDIAL or XCMS) processed on the same files
tools:
- name: Mass-Suite (MSS)
  role: Primary feature extractor whose output is matched against reference tools
    in m/z–RT space (Euclidean-distance alignment)
  repo: https://github.com/XiminHu/mass-suite
- name: MSDIAL
  role: Reference open-source platform providing comparison feature lists (target
    overlap ~52 ± 5%)
- name: XCMS
  role: Reference open-source platform providing comparison feature lists (target
    overlap ~52 ± 6%)
- name: pymzML
  role: Parses .mzML raw files within MSS to produce the Python-compatible scan metadata
    that feeds peak picking before cross-tool matching
  repo: https://github.com/pymzml/pymzML
- name: Python / Pandas
  role: Environment and dataframe library used to load feature lists, match shared
    features, and compute overlap statistics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:eae0384a0d10d442fab2168fb6d372b6b989767082cab98f7978fc98282bf7e7
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
  - pymzML
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cross-platform-feature-matching

## Summary

Quantifies how many features (m/z–RT pairs) extracted by MSS are recovered by reference open-source HRMS platforms (MSDIAL, XCMS) run on the same .mzML files, computing overlap fractions on mixed chemical standards of known composition as a cross-platform validation.

## When to use

Use when validating a new/in-house HRMS feature-extraction workflow by measuring the fraction of its m/z–RT features recovered by reference tools like MSDIAL or XCMS on identical mzML files of samples with known composition.

## When NOT to use

- Input is still raw .mzML data rather than exported per-tool feature lists — first run each platform's feature extraction and export its feature table before matching
- The goal is absolute extraction accuracy (recovery of the 398 known ENTACT peaks, 99.5% accuracy) rather than cross-tool overlap — that is a separate check against ground-truth composition
- Only one platform's output is available — overlap fractions require MSS and at least one reference tool (MSDIAL or XCMS) processed on the same files

## Inputs

- MSS-extracted feature lists (m/z, RT, peak area) per ENTACT mixture sample (#505, #506, #508)
- Exported feature tables from MSDIAL run on the same .mzML files with matched peak-picking parameters
- Exported feature tables from XCMS run on the same .mzML files with matched peak-picking parameters

## Outputs

- Per-sample overlap fractions for MSS-vs-MSDIAL and MSS-vs-XCMS
- Aggregated mean overlap ± standard deviation per comparison platform (validation targets ~52% for both)
- overlap_fractions.csv with per-sample and aggregated overlap fractions

## How to apply

Run MSS, MSDIAL, and XCMS on the same converted .mzML files (e.g., EPA ENTACT mixtures #505, #506, #508 containing 398 MS-amenable chemicals), with peak-picking parameters matched as closely as possible across platforms. Load each platform's exported feature list (m/z–RT pairs with peak areas) into pandas DataFrames. Match features between the MSS list and the reference list by proximity in m/z–RT space, consistent with MSS's Euclidean-distance alignment, per sample and per comparison platform. Compute the overlap fraction per sample as shared features divided by the MSS-reported feature count, then aggregate across samples as mean overlap ± standard deviation (expected: 52 ± 5% for MSDIAL, 52 ± 6% for XCMS). An independent sanity anchor is that MSS itself recovers 99.5% of the 398 known peaks (2 unmatched), so the denominator is trustworthy before any overlap is computed.

## Related tools

- **Mass-Suite (MSS)** (Primary feature extractor whose output is matched against reference tools in m/z–RT space (Euclidean-distance alignment)) — https://github.com/XiminHu/mass-suite
- **MSDIAL** (Reference open-source platform providing comparison feature lists (target overlap ~52 ± 5%))
- **XCMS** (Reference open-source platform providing comparison feature lists (target overlap ~52 ± 6%))
- **pymzML** (Parses .mzML raw files within MSS to produce the Python-compatible scan metadata that feeds peak picking before cross-tool matching) — https://github.com/pymzml/pymzML
- **Python / Pandas** (Environment and dataframe library used to load feature lists, match shared features, and compute overlap statistics)

## Examples

```
from scipy.spatial.distance import cdist; shared = (cdist(mss_ft[['mz','rt']], msdial_ft[['mz','rt']]) < tol).any(axis=1).sum(); overlap = shared / len(mss_ft)
```

## Evaluation signals

- Aggregated mean overlap is approximately 52 ± 5% for MSDIAL and 52 ± 6% for XCMS, matching the article-reported values
- MSS denominator is anchored on known composition: MSS extracted 99.5% of peaks (2 of 398 unmatched) present in all three ENTACT mixtures
- Per-sample overlap fractions are computed for each of samples #505, #506, and #508 and written to overlap_fractions.csv alongside the aggregates
- Feature matching uses proximity in m/z–RT space (Euclidean-distance alignment as in MSS), not exact string/numeric equality
- Peak-picking parameters on MSDIAL/XCMS were matched as closely as possible across platforms before comparison

## Limitations

- Overlap denominators and fractions depend on matched-but-not-identical peak-picking parameter settings across platforms; only 'matched as closely as possible' equivalence is achievable
- The MSS repository publishes no version number, DOI/Zenodo-archived release, changelog, or pinned dependency versions, so pinning the exact code state behind the 52% benchmark is difficult
- Supplementary material (Additional file 1) with experimental details and processing parameters is referenced but not available in the extracted context

## Evidence

- [methods] On the validation dataset, the fraction of MSS-extracted features overlapping other open-source tools averaged 52 ± 5% for MSDIAL and 52 ± 6% for XCMS, validating MSS performance.: "on average 52 ± 5% and 52 ± 6%, for MSDIAL and XCMS respectively), validating MSS performance"
- [methods] MSS anchoring: near-complete recovery of known peaks in the ENTACT mixtures.: "MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures"
- [abstract] Headline abstract claim motivating the overlap computation.: "and ~ 52% overlap of extracted features relative to other open-source software tools"
- [readme] README changelog confirming MSDIAL comparison was the validation route.: "2020/09/01: Feature extraction pipeline validated comparing with msdial pipeline output"
- [readme] README on alignment, the mechanism mirrored when matching MSS features to reference-tool features.: "Data from different experiments will be aligned together according to user defined error range."
