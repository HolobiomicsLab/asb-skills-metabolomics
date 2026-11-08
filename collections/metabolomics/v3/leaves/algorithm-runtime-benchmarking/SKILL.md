---
name: algorithm-runtime-benchmarking
description: Use when several candidate peak-picking algorithms must be compared for
  reliability vs. wall-clock runtime on LC-HRMS chromatogram data (mzXML), e.g. to
  choose a MetMatch peak-picking backend.
when_to_use_negative:
- The input is already a processed feature table rather than raw chromatogram data
  — peak picking (and therefore this benchmark) is unnecessary
- Only a single peak-picking algorithm is available, so no runtime/reliability comparison
  can be made
- The task is metabolite alignment of feature lists, not peak-picking algorithm selection
  or benchmarking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetMatch
  role: Source of the three peak-picking algorithms (MassSpecWavelet, Gaussian peak
    correlation, Savitzky-Golay filter) and the example mzXML reference chromatogram
    used in the benchmark
  repo: github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Runtime required to run MetMatch and to parse the mzXML reference chromatogram
    during the benchmark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:c5326202cb0dcca415fadbcd21ad5725cd6a1d044a27ab5355067ae93a6266f9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java JRE 1.8
  - MetMatch
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# algorithm-runtime-benchmarking

## Summary

Benchmark the runtimes of alternative chromatographic peak-picking algorithms (MassSpecWavelet, Gaussian peak correlation, Savitzky-Golay filter) implemented in MetMatch, which operates on LC-HRMS chromatograms stored as mzXML and requires Java JRE 1.8 or newer.

## When to use

Use when several candidate peak-picking algorithms must be compared for reliability vs. wall-clock runtime on LC-HRMS chromatogram data (mzXML), e.g. to choose a MetMatch peak-picking backend.

## When NOT to use

- The input is already a processed feature table rather than raw chromatogram data — peak picking (and therefore this benchmark) is unnecessary
- Only a single peak-picking algorithm is available, so no runtime/reliability comparison can be made
- The task is metabolite alignment of feature lists, not peak-picking algorithm selection or benchmarking

## Inputs

- Reference LC-HRMS chromatogram in mzXML format (MetMatch repository example file)
- MetMatch peak-picking algorithm specifications (Section 2.4, supplementary Section 1.1)
- Narrowed m/z and retention-time search frames
- Supplementary Table S1 reference runtime figures (Intel Core 2 Duo, 2.8 GHz)

## Outputs

- peak_picking_results.csv with one non-empty peak list (m/z, retention time, peak area) per algorithm
- Per-algorithm wall-clock runtime measurements

## How to apply

Load the reference chromatogram from the MetMatch repository's example mzXML file using a Java (JRE 1.8+) mzXML parser. Reimplement each of the three peak-picking algorithms described in Section 2.4 (details in supplementary Section 1.1), restricting each search to its narrowed m/z and retention-time frame, since MetMatch selects among MassSpecWavelet, Gaussian peak correlation, and Savitzky-Golay filter methods for peak picking within narrowed search frames. Run every algorithm independently over the full reference chromatogram and record the detected peaks (m/z, retention time, peak area) into a per-algorithm peak list, measuring the wall-clock runtime of each during execution. Write all peak lists and runtimes to peak_picking_results.csv and sanity-check the measured runtimes against the supplementary Table S1 figures obtained on an Intel Core 2 Duo (2.8 GHz), expecting MassSpecWavelet to be very reliable but significantly slower than the two alternatives.

## Related tools

- **MetMatch** (Source of the three peak-picking algorithms (MassSpecWavelet, Gaussian peak correlation, Savitzky-Golay filter) and the example mzXML reference chromatogram used in the benchmark) — github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Runtime required to run MetMatch and to parse the mzXML reference chromatogram during the benchmark)

## Evaluation signals

- peak_picking_results.csv exists and contains one non-empty peak list per algorithm
- Each peak list contains m/z, retention time, and peak area for the detected peaks
- Reported runtimes are of the same order of magnitude as the Table S1 reference timings (Intel Core 2 Duo, 2.8 GHz)
- MassSpecWavelet runtime is significantly longer than the Gaussian correlation and Savitzky-Golay runtimes, matching the article's finding
- All runs used the identical reference chromatogram and the same narrowed m/z and retention-time frames per algorithm

## Limitations

- Runtime figures are hardware-dependent; Table S1 reference timings were measured on an Intel Core 2 Duo (2.8 GHz), so direct numeric comparison requires similar-order hardware
- MetMatch requires Java runtime (JRE) version 1.8 or newer
- Missing signals note: discussion section content was not available in the extracted document, so caveats stated there could not be consulted

## Evidence

- [full_text] MetMatch offers the MassSpecWavelet peak-picking algorithm, which is very reliable but has significantly longer runtimes, alongside two alternative algorithms based on Gaussian peak correlation and the Savitzky-Golay filter.: [evidence span withheld — anchor status: not_found]
- [intro] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [abstract] Here, a novel software tool for the semi-automated alignment of different measurement sequences is presented.: "Here, a novel software tool for the semi-automated alignment of different measurement sequences is presented."
- [readme] MetMatch is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.: "MetMatch is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE."
