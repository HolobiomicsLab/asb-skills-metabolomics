---
name: mz-rt-pair-alignment
description: Use when two or more HRMS feature lists (m/z–RT pairs with peak areas,
  e.g., exported from MSS, MSDIAL, or XCMS on the same .mzML files) must be matched
  by proximity in retention-time and m/z space to compute shared-feature counts or
  overlap fractions.
when_to_use_negative:
- The input is already an aligned, merged feature table and the task is annotation
  or MS/MS fragment matching rather than cross-list feature matching.
- The goal is grouping features into individual chemicals (isotopes, adducts, in-source
  fragments) — the article lists feature grouping as future work, not implemented
  in MSS.
- The comparison lists were produced with mismatched peak-picking parameters on different
  raw files, making overlap fractions uninterpretable.
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: Mass-Suite (MSS)
  role: Source of the Euclidean-distance m/z–RT alignment approach and of the MSS
    feature list being compared; also exports aligned tables to .csv/.xlsx
  repo: https://github.com/XiminHu/mass-suite
- name: pymzML
  role: Parses the .mzML raw files into Python-compatible scan metadata (mssmain.get_scans)
    upstream of feature extraction and alignment
  repo: https://github.com/pymzml/pymzML
- name: MSDIAL
  role: Open-source HRMS tool whose exported feature list serves as the reference
    for cross-tool overlap matching
- name: XCMS
  role: Open-source HRMS tool whose exported feature list serves as the second reference
    for cross-tool overlap matching
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-023-00741-9
    title: 'Mass-Suite: a novel open-source python package for high-resolution mass
      spectrometry data analysis'
  source_content_hash: sha256:6fef2e8c737ab89862970a5f1df556946a6b14300626e8cfdc553a6d2fcd25a5
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mz-rt-pair-alignment

## Summary

Match HRMS feature lists represented as m/z–retention time pairs by proximity in the m/z–RT space, either to align features across replicated experiments (MSS alignment module) or to quantify cross-tool feature overlap (e.g., MSS vs. MSDIAL/XCMS).

## When to use

Use when two or more HRMS feature lists (m/z–RT pairs with peak areas, e.g., exported from MSS, MSDIAL, or XCMS on the same .mzML files) must be matched by proximity in retention-time and m/z space to compute shared-feature counts or overlap fractions.

## When NOT to use

- The input is already an aligned, merged feature table and the task is annotation or MS/MS fragment matching rather than cross-list feature matching.
- The goal is grouping features into individual chemicals (isotopes, adducts, in-source fragments) — the article lists feature grouping as future work, not implemented in MSS.
- The comparison lists were produced with mismatched peak-picking parameters on different raw files, making overlap fractions uninterpretable.

## Inputs

- MSS-extracted feature lists (m/z–RT pairs with peak areas) for ENTACT mixture samples #505, #506, #508
- Reference feature lists exported from MSDIAL and XCMS run on the same .mzML files
- A user-defined alignment error range (m/z and RT tolerance)

## Outputs

- Per-sample shared-feature counts and overlap fractions (shared features / MSS-reported feature count)
- Aggregated mean overlap ± standard deviation per comparison tool (target: ~52% for both MSDIAL and XCMS)
- overlap_fractions.csv with per-sample and aggregated values

## How to apply

Load each feature list into a pandas DataFrame keyed on m/z–RT pairs with peak areas. Match features between lists using proximity in the m/z–RT space, consistent with the Euclidean-distance alignment used in MSS, where 'data from different experiments will be aligned together according to user defined error range'. To replicate the MSS-vs-other-tools benchmark, run MSDIAL and XCMS on the same .mzML files with peak-picking parameters matched as closely as possible, then for each ENTACT sample (#505, #506, #508) compute the overlap fraction as shared features divided by the MSS-reported feature count and aggregate the mean ± standard deviation per comparison tool. Validation expectations come from the source article: MSS itself recovered 99.5% of the 398 known ENTACT peaks, and cross-tool overlap averaged 52 ± 5% (MSDIAL) and 52 ± 6% (XCMS).

## Related tools

- **Mass-Suite (MSS)** (Source of the Euclidean-distance m/z–RT alignment approach and of the MSS feature list being compared; also exports aligned tables to .csv/.xlsx) — https://github.com/XiminHu/mass-suite
- **pymzML** (Parses the .mzML raw files into Python-compatible scan metadata (mssmain.get_scans) upstream of feature extraction and alignment) — https://github.com/pymzml/pymzML
- **MSDIAL** (Open-source HRMS tool whose exported feature list serves as the reference for cross-tool overlap matching)
- **XCMS** (Open-source HRMS tool whose exported feature list serves as the second reference for cross-tool overlap matching)

## Examples

```
pip install mass-suite; python -m mss_processor.py
```

## Evaluation signals

- Aggregated mean overlap should be approximately 52 ± 5% for MSDIAL and 52 ± 6% for XCMS, matching the article's reported values.
- As a sanity baseline, MSS itself should recover ~99.5% of the 398 known ENTACT peaks (only 2 unmatched) before cross-tool comparison.
- Per-sample overlap fractions must be computed as shared features divided by the MSS-reported feature count, then aggregated as mean ± SD across samples #505, #506, #508.
- Matched counts must not exceed the size of the smaller feature list, and each feature should match at most one partner under the proximity rule.
- Alignment tolerance must be explicit and identical across samples (MSS uses a user-defined error range); results are sensitive to this choice.

## Limitations

- MSS does not implement feature grouping (merging isotopes, adducts, in-source fragments) as individual chemicals — listed as future work — so overlap fractions count ungrouped features.
- Cross-tool overlap depends strongly on how closely peak-picking parameters are matched between platforms ('matched as closely as possible'); exact parameter equivalence is not achievable.
- No version number, archived release (Zenodo DOI), changelog, or pinned Python/dependency versions are stated for MSS, so exact reproduction may vary with package updates.
- The MSS alignment error range is user-defined; no single canonical tolerance is specified in the article.

## Evidence

- [methods] overlap fractions averaged ~52%: "on average 52 ± 5% and 52 ± 6%, for MSDIAL and XCMS respectively), validating MSS performance"
- [methods] MSS peak recovery baseline: "MSS extracted 99.5% of peaks (2 peaks out of 398 didn’t match) known to be present in all three mixtures"
- [readme] MSS alignment mechanism: "Data from different experiments will be aligned together according to user defined error range."
- [methods] Upstream mzML parsing: [evidence span withheld — anchor status: not_found]
