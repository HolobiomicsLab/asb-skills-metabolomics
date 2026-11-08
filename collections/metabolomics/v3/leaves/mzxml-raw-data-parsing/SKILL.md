---
name: mzxml-raw-data-parsing
description: Use when the input is raw LC-HRMS measurement data in mzXML format and
  the analysis goal requires extracting per-scan m/z and retention-time information
  (e.g., before peak picking with MetMatch's MassSpecWavelet, Gaussian correlation,
  or Savitzky-Golay algorithms).
when_to_use_negative:
- The input is already a processed feature list or peak table (m/z + retention time
  + peak area) rather than raw mzXML chromatogram data
- The goal is alignment/correction of m/z and retention time shifts between existing
  feature lists — that is MetMatch's downstream alignment task, not raw-data parsing
- The data format is not mzXML (e.g., vendor formats or mzML without conversion)
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetMatch
  role: Provides the example mzXML reference chromatogram and consumes parsed chromatogram
    data for peak picking and LC-HRMS alignment; implemented in Java and requiring
    JRE 1.8+
  repo: github.com/S-T-K/MetMatch
- name: Java JRE 1.8
  role: Runtime environment required to execute the Java-based mzXML parser and MetMatch
    itself
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo6040039
    title: 'MetMatch: A Semi-Automated Software Tool for the Comparison and Alignment
      of LC-HRMS Data from Different Metabolomics Experiments'
  source_content_hash: sha256:ae1ace925c9e336ea4dfdf986f95d3ce444a1e065104b0d279e625ae58bbfbd3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java JRE 1.8
  - MetMatch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mzxml-raw-data-parsing

## Summary

Parse raw LC-HRMS chromatogram data from mzXML files using a Java-based parser (JRE 1.8+, as required by MetMatch) to expose m/z values, retention times, and chromatographic peak information for downstream peak-picking and alignment workflows.

## When to use

Use when the input is raw LC-HRMS measurement data in mzXML format and the analysis goal requires extracting per-scan m/z and retention-time information (e.g., before peak picking with MetMatch's MassSpecWavelet, Gaussian correlation, or Savitzky-Golay algorithms).

## When NOT to use

- The input is already a processed feature list or peak table (m/z + retention time + peak area) rather than raw mzXML chromatogram data
- The goal is alignment/correction of m/z and retention time shifts between existing feature lists — that is MetMatch's downstream alignment task, not raw-data parsing
- The data format is not mzXML (e.g., vendor formats or mzML without conversion)

## Inputs

- Raw LC-HRMS chromatogram file in mzXML format (e.g., the example mzXML file from the MetMatch repository)
- Optionally, narrowed m/z and retention-time search frames limiting the region to parse

## Outputs

- Parsed chromatogram records with m/z, retention time, and peak area/intensity per feature
- Per-algorithm peak lists (e.g., for MassSpecWavelet, Gaussian peak correlation, Savitzky-Golay peak picking) that can be exported to CSV

## How to apply

Load the mzXML file (e.g., the reference chromatogram shipped as an example mzXML file in the MetMatch repository at github.com/S-T-K/MetMatch) with an mzXML parser running under Java JRE 1.8 or newer, since MetMatch explicitly requires JRE 1.8+. Restrict parsing to the relevant narrowed m/z and retention-time frames when the data feeds a peak-picking step, then emit structured records containing m/z, retention time, and intensity/peak area per scan or detected feature. Parsed chromatograms serve as the reference or comparison input for MetMatch's semi-automated alignment of LC-HRMS measurement sequences. Validate the parse by checking that detected peak lists are non-empty and that retention-time and m/z ranges are consistent with the instrument acquisition.

## Related tools

- **MetMatch** (Provides the example mzXML reference chromatogram and consumes parsed chromatogram data for peak picking and LC-HRMS alignment; implemented in Java and requiring JRE 1.8+) — github.com/S-T-K/MetMatch
- **Java JRE 1.8** (Runtime environment required to execute the Java-based mzXML parser and MetMatch itself)

## Evaluation signals

- peak_picking_results.csv exists and contains one non-empty peak list per peak-picking algorithm applied to the parsed chromatogram
- Parsed records cover the expected m/z and retention-time ranges of the reference chromatogram without dropped scans
- Peak lists include the expected triple (m/z, retention time, peak area) per detected peak
- The parser runs under JRE 1.8 or newer without runtime errors, matching MetMatch's stated requirement

## Limitations

- MetMatch requires Java runtime (JRE) version 1.8 or newer; older runtimes will fail
- The skill is specific to mzXML-formatted raw data; no other raw formats are described in the article or README
- The article's discussion-section content is not available, so format caveats beyond mzXML cannot be confirmed from context

## Evidence

- [readme] MetMatch requires Java runtime (JRE) version 1.8 or newer.: "MetMatch requires Java runtime (JRE) version 1.8 or newer."
- [other] Load the reference chromatogram from the MetMatch repository's example mzXML file using a Java (JRE 1.8+) mzXML parser.: [evidence span withheld — anchor status: not_found]
- [abstract] Due to its unsurpassed sensitivity and selectivity, LC-HRMS is one of the major analytical techniques in metabolomics research.: "Due to its unsurpassed sensitivity and selectivity, LC-HRMS is one of the major analytical techniques in metabolomics research."
- [intro] MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments.: "MetMatch is a software tool for the semi-automated comparison of different datasets of untargeted metabolomics experiments."
