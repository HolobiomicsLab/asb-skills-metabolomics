---
name: peak-picking-algorithm-reimplementation
description: Use when reproducing or benchmarking LC-HRMS chromatographic peak picking
  from MetMatch, comparing MassSpecWavelet vs. Gaussian-correlation vs. Savitzky-Golay
  runtimes, or porting MetMatch's Section 2.4 peak-picking logic to another language.
when_to_use_negative:
- The input is already a processed feature list or peak table (MetMatch takes such
  a reference feature list as input to alignment, not to peak picking)
- The goal is semi-automated alignment/correction of m/z and retention time shifts
  between datasets rather than reimplementation or benchmarking of the peak-picking
  algorithms themselves
- The chromatogram data is not in mzXML format and cannot be parsed by the Java (JRE
  1.8+) parser used here
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetMatch
  role: Source software whose three selectable peak-picking algorithms (MassSpecWavelet,
    Gaussian peak correlation, Savitzky-Golay) are reimplemented and benchmarked;
    its example mzXML provides the reference chromatogram
  repo: github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Required runtime to parse the MetMatch example mzXML reference chromatogram
    and to run/reference the Java-based algorithm implementations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:3e132ed359266fd6965a82a0611d45e1ca89fcbe8cdbd0f888210e1b5a01a5f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java JRE 1.8
  - MetMatch
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# peak-picking-algorithm-reimplementation

## Summary

Reimplements and benchmarks the three chromatographic peak-picking algorithms offered by MetMatch (MassSpecWavelet, Gaussian peak correlation, and Savitzky-Golay filter) so their detected peaks and runtimes can be compared, reflecting the paper's finding that MassSpecWavelet is very reliable but has significantly longer runtimes than the two alternatives.

## When to use

Use when reproducing or benchmarking LC-HRMS chromatographic peak picking from MetMatch, comparing MassSpecWavelet vs. Gaussian-correlation vs. Savitzky-Golay runtimes, or porting MetMatch's Section 2.4 peak-picking logic to another language.

## When NOT to use

- The input is already a processed feature list or peak table (MetMatch takes such a reference feature list as input to alignment, not to peak picking)
- The goal is semi-automated alignment/correction of m/z and retention time shifts between datasets rather than reimplementation or benchmarking of the peak-picking algorithms themselves
- The chromatogram data is not in mzXML format and cannot be parsed by the Java (JRE 1.8+) parser used here

## Inputs

- Reference chromatogram as an mzXML file (e.g. the example mzXML shipped with the MetMatch repository)
- Peak-picking algorithm specifications from MetMatch Section 2.4 and supplementary Section 1.1
- Narrowed m/z and retention-time search frames per peak to be picked
- Supplementary Table S1 reference runtime figures (Intel Core 2 Duo, 2.8 GHz) for sanity-checking

## Outputs

- peak_picking_results.csv containing one non-empty peak list per algorithm (m/z, retention time, peak area)
- Per-algorithm wall-clock runtimes recorded alongside the peak lists

## How to apply

Load the reference chromatogram from the MetMatch repository's example mzXML file using a Java (JRE 1.8+) mzXML parser. Reimplement each of the three peak-picking algorithms described in Section 2.4 of the MetMatch paper (with details in supplementary Section 1.1): MassSpecWavelet, Gaussian peak correlation, and Savitzky-Golay filtering. Restrict each algorithm's search to its narrowed m/z and retention-time frame, then run each independently over the full reference chromatogram to produce a per-algorithm peak list of m/z values, retention times, and peak areas. Measure wall-clock runtime during each execution and compare against the supplementary Table S1 reference timings obtained on an Intel Core 2 Duo (2.8 GHz). Expect MassSpecWavelet to detect peaks very reliably but to run significantly longer than the Gaussian-correlation and Savitzky-Golay based algorithms, which is the rationale for offering selectable algorithms.

## Related tools

- **MetMatch** (Source software whose three selectable peak-picking algorithms (MassSpecWavelet, Gaussian peak correlation, Savitzky-Golay) are reimplemented and benchmarked; its example mzXML provides the reference chromatogram) — github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Required runtime to parse the MetMatch example mzXML reference chromatogram and to run/reference the Java-based algorithm implementations)

## Evaluation signals

- peak_picking_results.csv exists and contains one non-empty peak list for each of the three algorithms (MassSpecWavelet, Gaussian correlation, Savitzky-Golay)
- Each peak list records m/z, retention time, and peak area for every detected peak
- Measured wall-clock runtimes are of the same order of magnitude as the supplementary Table S1 figures obtained on an Intel Core 2 Duo (2.8 GHz)
- The MassSpecWavelet implementation detects peaks very reliably relative to the two filter-based alternatives, consistent with the paper's reported runtime/reliability trade-off
- Each algorithm was run independently over the full reference chromatogram with searches restricted to the specified narrowed m/z and retention-time frames

## Limitations

- MassSpecWavelet peak picking is reliable but has significantly longer runtimes, which motivated MetMatch offering the two faster alternatives
- Runtime comparability depends on hardware: the reference Table S1 timings were measured on an Intel Core 2 Duo (2.8 GHz), so absolute times on other machines will differ (only order-of-magnitude agreement is expected)
- Algorithm details are partly deferred to supplementary Section 1.1, so reimplementation accuracy depends on faithful reproduction of those specifications

## Evidence

- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [intro] MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments.: "MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments."
- [results] MetMatch offers MassSpecWavelet peak picking alongside Gaussian correlation and Savitzky-Golay alternatives.: [evidence span withheld — anchor status: not_found]
