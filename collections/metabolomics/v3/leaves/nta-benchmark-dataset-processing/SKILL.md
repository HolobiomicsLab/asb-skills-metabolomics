---
name: nta-benchmark-dataset-processing
description: Use when validating a new HRMS feature-extraction pipeline against established
  open-source tools (MSDIAL, XCMS) on mixed chemical standards of known composition,
  such as EPA ENTACT mixtures, where expected true peaks are enumerated.
when_to_use_negative:
- The goal is quantitative annotation or MS/MS fragment identification rather than
  extraction-performance benchmarking — use fragment searching against the MoNA MassBank
  database instead.
- The sample mixture has no known chemical composition, so there is no ground-truth
  peak count to interpret overlap fractions against.
- The input is already a single curated feature table from one platform with no second
  tool's output to compare against.
edam_topics:
- http://edamontology.org/topic_3364
tools:
- name: Mass-Suite (MSS)
  role: Candidate extractor whose feature output (from mssmain.get_scans + mssmain.peak_pick
    on .mzML) is benchmarked against reference tools
  repo: https://github.com/XiminHu/mass-suite
- name: MSDIAL
  role: Reference open-source HRMS tool producing the comparison feature list for
    overlap computation
- name: XCMS
  role: Reference open-source HRMS tool producing the comparison feature list for
    overlap computation
- name: pymzML
  role: Python parser that reads the .mzML benchmark files for import into MSS
  repo: https://github.com/pymzml/pymzML
- name: Python/Pandas
  role: Loads, aligns, and aggregates the feature lists used to compute overlap fractions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:7f5bbd7f5979638fd8abd9160fe896268fc9166a6f8172cc3169404a536bcae7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiminHu/mass-suite
  tools:
  - MSDIAL
  - Mass-Suite (MSS)
  - Pandas
  - Python
  - Python/Pandas
  - XCMS
  - pymzML
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3364
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# nta-benchmark-dataset-processing

## Summary

A cross-platform feature-overlap benchmark for HRMS non-targeted analysis: quantify what fraction of MSS-extracted features (m/z–RT pairs) are recovered by MSDIAL and XCMS on EPA ENTACT mixtures of known composition, yielding ~52% overlap figures that validate a new extraction pipeline.

## When to use

Use when validating a new HRMS feature-extraction pipeline against established open-source tools (MSDIAL, XCMS) on mixed chemical standards of known composition, such as EPA ENTACT mixtures, where expected true peaks are enumerated.

## When NOT to use

- The goal is quantitative annotation or MS/MS fragment identification rather than extraction-performance benchmarking — use fragment searching against the MoNA MassBank database instead.
- The sample mixture has no known chemical composition, so there is no ground-truth peak count to interpret overlap fractions against.
- The input is already a single curated feature table from one platform with no second tool's output to compare against.

## Inputs

- MSS-extracted feature lists (m/z–RT pairs with peak areas) from MSS peak_pick on ENTACT mixture samples #505, #506 and #508
- Reference feature tables exported from MSDIAL run on the same .mzML files
- Reference feature tables exported from XCMS run on the same .mzML files
- .mzML converted raw HRMS files for the three ENTACT mixtures

## Outputs

- Per-sample overlap fractions (shared features / MSS feature count) for MSDIAL and XCMS separately
- Aggregated mean overlap ± standard deviation per comparison platform (~52 ± 5% MSDIAL, ~52 ± 6% XCMS)
- overlap_fractions.csv containing per-sample and aggregated overlap fractions

## How to apply

Run the candidate extractor (Mass-Suite: mssmain.get_scans on .mzML input via pymzML, then mssmain.peak_pick with peak width filter, replicated peak filter, and regression-based peak boundary determination) on the benchmark samples ENTACT mixtures #505, #506 and #508 (398 MS-amenable chemicals). Export feature lists (m/z, retention time, peak area) from MSDIAL and XCMS run on the identical .mzML files, with peak-picking parameters matched as closely as possible across platforms. For each sample and each reference tool, match features between the MSS list and the reference list by proximity in m/z–RT space (consistent with MSS's Euclidean-distance alignment) and count shared features. Compute the per-sample overlap fraction as shared features divided by the MSS-reported feature count, then report the mean ± standard deviation across the three samples, writing per-sample and aggregated values to overlap_fractions.csv. Correct application reproduces mean overlaps of approximately 52 ± 5% for MSDIAL and 52 ± 6% for XCMS.

## Related tools

- **Mass-Suite (MSS)** (Candidate extractor whose feature output (from mssmain.get_scans + mssmain.peak_pick on .mzML) is benchmarked against reference tools) — https://github.com/XiminHu/mass-suite
- **MSDIAL** (Reference open-source HRMS tool producing the comparison feature list for overlap computation)
- **XCMS** (Reference open-source HRMS tool producing the comparison feature list for overlap computation)
- **pymzML** (Python parser that reads the .mzML benchmark files for import into MSS) — https://github.com/pymzml/pymzML
- **Python/Pandas** (Loads, aligns, and aggregates the feature lists used to compute overlap fractions)

## Examples

```
pip install mass-suite; import mss  # then run mssmain.get_scans/peak_pick on ENTACT #505/#506/#508 .mzML files and compute overlap vs MSDIAL/XCMS exports
```

## Evaluation signals

- Aggregated mean overlap is approximately 52 ± 5% for MSDIAL and 52 ± 6% for XCMS across ENTACT samples #505, #506 and #508
- MSS extracts ~99.5% of known peaks (397 of 398 matched across the three mixtures) before overlap computation
- Per-sample overlap fractions are computed for all three samples and each reference platform, with standard deviations ≤ 6 points
- All per-sample and aggregated fractions are written to overlap_fractions.csv without missing platform/sample rows
- Feature matching uses m/z–RT proximity consistent with MSS's Euclidean-distance alignment, not exact string equality of m/z

## Limitations

- Peak-picking parameters between platforms can only be matched 'as closely as possible' — residual algorithmic differences mean ~48% of MSS features lack cross-tool counterparts even on standards of known composition.
- MSS feature grouping (merging isotopes, adducts, in-source fragments) is not yet implemented, so features that are chemically the same compound may inflate or deflate overlap counts.
- The package is actively maintained with no pinned version, changelog, or archived DOI release; overlap results are sensitive to which MSS version is used.
- Benchmark conclusions depend on supplementary data (Additional file 1: Text S2) describing the ENTACT mixtures, which may not be directly accessible.

## Evidence

- [methods] overlap 52±5%: "on average 52 ± 5% and 52 ± 6%, for MSDIAL and XCMS respectively), validating MSS performance"
- [methods] 99.5% extraction recall: [evidence span withheld — anchor status: not_found]
- [readme] README validation precedent: "2020/09/01: Feature extraction pipeline validated comparing with msdial pipeline output"
- [other] workflow matching rule: [evidence span withheld — anchor status: not_found]
- [methods] benchmark samples: [evidence span withheld — anchor status: not_found]
