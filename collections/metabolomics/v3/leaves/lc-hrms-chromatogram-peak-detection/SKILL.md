---
name: lc-hrms-chromatogram-peak-detection
description: Use when raw LC-HRMS chromatograms (e.g., mzXML files) must be converted
  into peak lists (m/z, retention time, peak area) before metabolomics alignment,
  and algorithm speed/reliability trade-offs matter.
when_to_use_negative:
- Input is already a processed feature list or peak table (MetMatch's alignment stage
  expects to start from raw chromatographic data)
- The task is retention-time or m/z shift correction of existing features rather than
  initial peak detection from chromatograms
- The runtime environment cannot provide Java JRE 1.8 or newer, which MetMatch requires
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MetMatch
  role: Software tool providing the three selectable chromatographic peak-picking
    algorithms (MassSpecWavelet, Gaussian peak correlation, Savitzky-Golay) applied
    within narrowed search frames
  repo: github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Required runtime (JRE 1.8 or newer) to execute MetMatch and its mzXML parsing
    / peak-picking code
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:f246d679facffd5ca81a163a481ea05233ac7cb374e78696f982c5990a1fb169
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java JRE 1.8
  - MetMatch
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# lc-hrms-chromatogram-peak-detection

## Summary

Peak picking of LC-HRMS chromatograms using MetMatch's selectable peak-picking algorithms (MassSpecWavelet, Gaussian peak correlation, and Savitzky-Golay filter), applied within narrowed m/z and retention-time search frames.

## When to use

Use when raw LC-HRMS chromatograms (e.g., mzXML files) must be converted into peak lists (m/z, retention time, peak area) before metabolomics alignment, and algorithm speed/reliability trade-offs matter.

## When NOT to use

- Input is already a processed feature list or peak table (MetMatch's alignment stage expects to start from raw chromatographic data)
- The task is retention-time or m/z shift correction of existing features rather than initial peak detection from chromatograms
- The runtime environment cannot provide Java JRE 1.8 or newer, which MetMatch requires

## Inputs

- LC-HRMS chromatogram file (e.g., example mzXML from the MetMatch repository)
- Narrowed m/z and retention-time search frames

## Outputs

- Per-algorithm peak list (m/z, retention time, peak area)
- Measured wall-clock runtime per peak-picking algorithm (e.g., written to peak_picking_results.csv)

## How to apply

Load the reference chromatogram from an mzXML file using a Java (JRE 1.8+) mzXML parser, as required by MetMatch. Select one of the three implemented peak-picking algorithms — MassSpecWavelet (most reliable but significantly slower), Gaussian peak correlation, or Savitzky-Golay filter — and restrict the peak search to a narrowed m/z and retention-time frame. Run the chosen algorithm over the full reference chromatogram and record each detected peak's m/z, retention time, and peak area into a per-algorithm peak list. Measure wall-clock runtime during execution and benchmark it against published reference timings (supplementary Table S1, obtained on an Intel Core 2 Duo at 2.8 GHz).

## Related tools

- **MetMatch** (Software tool providing the three selectable chromatographic peak-picking algorithms (MassSpecWavelet, Gaussian peak correlation, Savitzky-Golay) applied within narrowed search frames) — github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Required runtime (JRE 1.8 or newer) to execute MetMatch and its mzXML parsing / peak-picking code)

## Examples

```
java -jar MetMatch.jar  # launch MetMatch GUI (requires JRE 1.8+), load the reference chromatogram mzXML, and run a selected peak-picking algorithm within a narrowed m/z and RT frame
```

## Evaluation signals

- peak_picking_results.csv exists and contains one non-empty peak list (m/z, retention time, peak area) per algorithm
- Measured runtimes are of the same order of magnitude as the supplementary Table S1 reference timings (Intel Core 2 Duo, 2.8 GHz)
- MassSpecWavelet yields the most reliable peak detection, while the Gaussian-correlation and Savitzky-Golay variants run substantially faster
- Detected peaks fall within the narrowed m/z and retention-time search frames applied to the reference chromatogram

## Limitations

- The MassSpecWavelet algorithm, while very reliable, has significantly longer runtimes than the two alternative algorithms
- MetMatch is a semi-automated GUI tool implemented in Java and cannot be executed without JRE 1.8 or newer
- MetMatch is distributed under GPLv2+ WITHOUT ANY WARRANTY; issue reports and improvements must go through the repository's issue tracker

## Evidence

- [other] MassSpecWavelet reliability vs. runtime trade-off: [evidence span withheld — anchor status: not_found]
- [readme] MetMatch purpose: "MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments. It provides methods to correct LC-HRMS data exhibiting m/z and retention"
- [intro] Java runtime requirement: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
