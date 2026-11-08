---
name: feature-detection-in-mzml-acquisitions
description: Use when raw Thermo .raw or .mzML flow injection MSn files must be turned
  into per-file feature/compound detections in mzmine, e.g. to count molecules detected
  in positive-only, negative-only, or both ionization modes.
when_to_use_negative:
- The input is already a feature table or peak list — re-running feature detection
  would duplicate work and risk inconsistent identifiers.
- The goal is direct compound identification from MS/MS spectra without per-file feature
  counting across ionization modes.
- Vendor .raw files are required without conversion and no ThermoRawFileParser (or
  mzmine-compatible) reader is available.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ThermoRawFileParser
  role: Converts Thermo .raw files to MGF, mzML or indexed mzML so mzmine can process
    the flow injection acquisitions.
  repo: https://github.com/CompOmics/ThermoRawFileParser
- name: mzmine
  role: Runs the high-throughput feature detection and processing pipeline over the
    converted mzML files to produce per-file feature lists.
  repo: https://github.com/mzmine/mzmine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:6bba3010651b613e205dabf7578149f0225376459d8c9615f80de738b6f45ab8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CompOmics/ThermoRawFileParser
  tools:
  - Orbitrap ID-X
  - ThermoRawFileParser
  - mzmine
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# feature-detection-in-mzml-acquisitions

## Summary

Mass-spectrometry feature detection within mzML/.raw flow-injection acquisitions (converted by ThermoRawFileParser) using mzmine to generate per-file compound feature lists that feed MSnLib's ionization-mode coverage statistics.

## When to use

Use when raw Thermo .raw or .mzML flow injection MSn files must be turned into per-file feature/compound detections in mzmine, e.g. to count molecules detected in positive-only, negative-only, or both ionization modes.

## When NOT to use

- The input is already a feature table or peak list — re-running feature detection would duplicate work and risk inconsistent identifiers.
- The goal is direct compound identification from MS/MS spectra without per-file feature counting across ionization modes.
- Vendor .raw files are required without conversion and no ThermoRawFileParser (or mzmine-compatible) reader is available.

## Inputs

- Thermo .raw acquisition files from Orbitrap ID-X (Xcalibur sequences) flow injection experiments
- ThermoRawFileParser-converted .mzML or indexed .mzML files
- Cleaned metadata table (from metadata_cleanup_prefect.py) with sample_id, plate_id, and well/vial location columns

## Outputs

- Per-file feature lists from mzmine covering each ionization mode
- Unique compound counts per ionization mode (positive-only, negative-only, both)
- detection_statistics.csv with the count breakdown table

## How to apply

Convert Thermo .raw acquisitions to open format with CompOmics/ThermoRawFileParser (to mzML or indexed mzML), since mzmine and downstream mzML processing pipelines work with open formats rather than vendor binaries. Load the converted mzML files into mzmine, the open-source MS data processing software that MSnLib builds its high-throughput acquisition and processing pipeline on, and run feature detection over the flow injection data per acquisition file. Assign each detected feature to a unique compound identifier by joining the mzmine feature lists with the cleaned metadata table produced by the metadata_cleanup_prefect.py flow. Compute per-ionization-mode statistics by counting unique compounds detected in positive mode only, negative mode only, and the intersection of the two; validate against MSnLib's reported values (30,008 unique compounds, >12,200 positive-only, >3,400 negative-only, ~14,300 in both modes) and export the breakdown to detection_statistics.csv. Metadata must follow the shared template with unique sample_id, plate_id and well/vial location columns so feature-to-compound joins are unambiguous.

## Related tools

- **ThermoRawFileParser** (Converts Thermo .raw files to MGF, mzML or indexed mzML so mzmine can process the flow injection acquisitions.) — https://github.com/CompOmics/ThermoRawFileParser
- **mzmine** (Runs the high-throughput feature detection and processing pipeline over the converted mzML files to produce per-file feature lists.) — https://github.com/mzmine/mzmine

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- Unique compound count matches the reported 30,008 for MSnLib's flow injection–Orbitrap data.
- Positive-only count exceeds 12,200 and negative-only count exceeds 3,400.
- Intersection of positive- and negative-mode detections is approximately 14,300 compounds.
- Sum of positive-only + negative-only + both-mode counts equals the total unique compound count.
- Every detected feature maps to a unique compound identifier after joining with the cleaned metadata table.

## Limitations

- ThermoRawFileParser versions before 2.0.0 require Mono to run on Linux and Mac.
- Feature detection quality depends on the Orbitrap ID-X sequence setup and a metadata sheet with unique sample id, plate name (batch identifier) as plate_id, and vial/well location — missing fields break the feature-to-compound join.
- Counts depend on which external compound databases were enabled; if a source (Dictionary of Natural Products, DrugBank, LOTUS) needs a local file and/or special access it must otherwise be set to False in jobs.py, changing the total detection statistics.

## Evidence

- [abstract] MSnLib reports >2 million spectra covering 30,008 unique small molecules: "a machine learning-ready open resource of >2 million spectra in MS n trees of 30,008 unique small molecules"
- [abstract] mzmine is the processing backbone: "built with a high-throughput data acquisition and processing pipeline in the open-source software mzmine"
- [readme] ThermoRawFileParser supports mzML and indexed mzML conversion: "A tool allowing reading Thermo RAW mass spectrometer files and converting to common open formats on all platforms supporting .NET Core.

Supported formats:
* MGF
* mzML and indexed mzML"
- [readme] Metadata template required for feature-to-compound joins: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
