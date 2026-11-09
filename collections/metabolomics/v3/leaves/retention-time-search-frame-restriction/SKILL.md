---
name: retention-time-search-frame-restriction
description: Use when re-detecting peaks from LC-HRMS mzXML chromatograms for a known
  reference feature list, and narrowed m/z + retention-time search windows are available
  or can be derived per feature.
when_to_use_negative:
- Input is already a processed feature table (m/z, retention time, peak area) and
  no raw chromatographic peak re-detection is needed
- Runtime budget is strict and the dataset is large — MassSpecWavelet is very reliable
  but has significantly longer runtimes than the Gaussian correlation or Savitzky-Golay
  alternatives
- Data is not LC-HRMS chromatographic data with m/z and retention time dimensions,
  since MetMatch's correction and alignment operate on LC-HRMS shifts relative to
  a reference chromatogram
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetMatch
  role: Provides the three peak-picking algorithms (MassSpecWavelet, Gaussian peak
    correlation, Savitzky-Golay filter) that are run within narrowed search frames
    on the reference chromatogram
  repo: github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Required runtime environment to load mzXML chromatograms and execute MetMatch's
    Java-based peak-picking routines
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:3b0111e8bf8872bf5aa51bdb20bd03a096dac97c0f5bb6a8e8ad66f0eddd2837
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java JRE 1.8
  - MetMatch
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# retention-time-search-frame-restriction

## Summary

Restrict MetMatch's chromatographic peak-picking search to narrowed m/z and retention-time frames around expected features in an LC-HRMS reference chromatogram, choosing between the MassSpecWavelet, Gaussian peak-correlation, and Savitzky-Golay filter algorithms while benchmarking their runtimes.

## When to use

Use when re-detecting peaks from LC-HRMS mzXML chromatograms for a known reference feature list, and narrowed m/z + retention-time search windows are available or can be derived per feature.

## When NOT to use

- Input is already a processed feature table (m/z, retention time, peak area) and no raw chromatographic peak re-detection is needed
- Runtime budget is strict and the dataset is large — MassSpecWavelet is very reliable but has significantly longer runtimes than the Gaussian correlation or Savitzky-Golay alternatives
- Data is not LC-HRMS chromatographic data with m/z and retention time dimensions, since MetMatch's correction and alignment operate on LC-HRMS shifts relative to a reference chromatogram

## Inputs

- mzXML file containing the reference LC-HRMS chromatogram (e.g. from the MetMatch repository's example data)
- Reference feature list with m/z values and retention times
- Narrowed m/z and retention-time search frame per feature

## Outputs

- Per-algorithm peak list with m/z, retention time, and peak area for each detected peak
- Measured wall-clock runtime per peak-picking algorithm
- Combined results file (peak_picking_results.csv) with peak lists and runtimes

## How to apply

Load the reference chromatogram from an mzXML file using a Java (JRE 1.8+) parser, as required by MetMatch. For each entry in the reference feature list (m/z values and retention times), define a narrowed m/z and retention-time search frame so that peak picking is restricted only to that window, reducing false detections outside the expected feature region. Run each candidate peak-picking algorithm independently over the full reference chromatogram within its frame: MassSpecWavelet (very reliable but with significantly longer runtimes), Gaussian peak correlation, and Savitzky-Golay filtering (Section 2.4; details in supplementary Section 1.1). Record detected peaks (m/z, retention time, peak area) into a per-algorithm peak list and measure each algorithm's wall-clock runtime. The frame restriction is the key decision point: tighter frames speed up search but must be wide enough to capture the shifted peaks being aligned relative to the reference chromatogram.

## Related tools

- **MetMatch** (Provides the three peak-picking algorithms (MassSpecWavelet, Gaussian peak correlation, Savitzky-Golay filter) that are run within narrowed search frames on the reference chromatogram) — github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Required runtime environment to load mzXML chromatograms and execute MetMatch's Java-based peak-picking routines)

## Evaluation signals

- peak_picking_results.csv exists and contains one non-empty peak list per algorithm (MassSpecWavelet, Gaussian correlation, Savitzky-Golay)
- Every detected peak's m/z and retention time falls inside its assigned narrowed search frame
- Measured runtimes are of the same order of magnitude as the supplementary Table S1 figures obtained on an Intel Core 2 Duo (2.8 GHz)
- Peak records include all three expected attributes: m/z, retention time, and peak area
- Cross-check that the most reliable algorithm (MassSpecWavelet) does not miss peaks found by the faster alternatives within the same frame

## Limitations

- MassSpecWavelet peak picking is very reliable but has significantly longer runtimes, making it costly for large batches
- MetMatch requires Java runtime (JRE) version 1.8 or newer; older environments cannot load mzXML data
- Search-frame boundaries are a decision point: frames too narrow may clip shifted peaks, since MetMatch exists to correct m/z and retention time shifts relative to a reference chromatogram

## Evidence

- [abstract] Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference: "Based on a feature list (i.e., processed LC-HRMS chromatograms with mass-to-charge ratio (m/z) values and retention times) that serves as a reference"
- [intro] It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram.: "It provides methods to correct LC-HRMS data exhibiting m/z and retention time shifts relative to a reference chromatogram."
- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [other] MetMatch offers the MassSpecWavelet peak-picking algorithm, which is very reliable but has significantly longer runtimes, alongside two alternative algorithms based on Gaussian peak correlation and the Savitzky-Golay filter.: [evidence span withheld — anchor status: not_found]
