---
name: glycan-identification-benchmark-validation
description: Use when an LC-MS/MS glycan identification method must be validated against
  a manually confirmed glycan list, requiring true/false positive counts and RT agreement
  from MS1 peak calling.
when_to_use_negative:
- The glycans of interest are O-linked or not present in the app's glycan databases
  — the app's scope is limited to fluorescent-labeled N-linked glycans and has no
  de novo identification capability.
- Input is MALDI-TOF data or raw vendor files that have not been converted to mzXML
  via ProteoWizard, or you need to process multiple mzXML files simultaneously — the
  app cannot process multiple mzXML files at once.
- You need customizable isomer peak RT ranges — the isomer peak retention time range
  is a fixed value, not user-adjustable.
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0080
tools:
- name: GlyKAn AZ
  role: MATLAB app that performs the MS1 peak identification and MS2 spectral confirmation
    being benchmarked; obtained from the AstraZeneca GitHub repository under MIT license
  repo: github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- name: ProteoWizard
  role: Converts raw Orbitrap LC-MS/MS vendor files to the mzXML format required by
    GlyKAn AZ
- name: MATLAB
  role: Runtime environment in which the GlyKAn AZ app is built and executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-023-05346-5
    title: A MATLAB-based app to improve LC–MS/MS data analysis for N-linked glycan
      peak identification
  source_content_hash: sha256:29c9cc01f0006d1c16bc989532bde0c92eb6d21de4fa646ef324ba7ae73b894a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GlyKAn AZ
  - MATLAB
  - OrbiTrap
  - ProteoWizard
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 20
---

# glycan-identification-benchmark-validation

## Summary

Benchmark a glycan identification workflow (e.g., GlyKAn AZ) against a manually curated LC-MS/MS glycan list, counting true/false positives and matching retention times. This validates that automated N-linked glycan peak identification reproduces expert analyst results.

## When to use

Use when an LC-MS/MS glycan identification method must be validated against a manually confirmed glycan list, requiring true/false positive counts and RT agreement from MS1 peak calling.

## When NOT to use

- The glycans of interest are O-linked or not present in the app's glycan databases — the app's scope is limited to fluorescent-labeled N-linked glycans and has no de novo identification capability.
- Input is MALDI-TOF data or raw vendor files that have not been converted to mzXML via ProteoWizard, or you need to process multiple mzXML files simultaneously — the app cannot process multiple mzXML files at once.
- You need customizable isomer peak RT ranges — the isomer peak retention time range is a fixed value, not user-adjustable.

## Inputs

- Raw Orbitrap LC-MS/MS files converted to mzXML format (via ProteoWizard)
- MS1 Database (glycan names, theoretical masses in Da, optional expected RTs in minutes)
- MS2 Fragment Database and MS2 fragment image repository
- Manually curated reference list of identified glycans with retention times (here: 20 glycans identified by analysts)

## Outputs

- MS1 output Excel file listing likely glycan identifications with retention times
- True positive / false positive counts against the manual glycan list (benchmark: 20 TP / 0 FP)
- Annotated and labeled MS2 fragmentation spectra for selected glycan-RT pairs
- Matched RT comparison between app-identified and manually identified glycans

## How to apply

Acquire the GlyKAn AZ app from github.com/ZaylaSchaeffer/GlyKAn-AZ-application together with its MS1 and MS2 glycan databases, and convert raw Orbitrap LC-MS/MS files to mzXML using ProteoWizard. Run the MS1 Peak ID tab on the mzXML plus the MS1 Database (glycan names, theoretical masses in Da, optional expected RTs in minutes) using the benchmarked settings for the monoclonal antibody dataset: 25% relative intensity threshold and 10 ppm mass accuracy threshold. Count true positives against the 20 manually identified glycans and check that false positives are zero. Then load the MS1 output, MS2 Fragment Database, and fragment image repository into the MS2 Peak Confirmation tab to generate annotated and labeled MS2 fragmentation spectra for the identified glycan-RT pairs. Correctness is judged by reproducing the paper's outcome: 20 true positives / 0 false positives, with the app identifying all 20 glycans at the same retention times as manual analysis.

## Related tools

- **GlyKAn AZ** (MATLAB app that performs the MS1 peak identification and MS2 spectral confirmation being benchmarked; obtained from the AstraZeneca GitHub repository under MIT license) — github.com/ZaylaSchaeffer/GlyKAn-AZ-application
- **ProteoWizard** (Converts raw Orbitrap LC-MS/MS vendor files to the mzXML format required by GlyKAn AZ)
- **MATLAB** (Runtime environment in which the GlyKAn AZ app is built and executed)

## Evaluation signals

- At 25% relative intensity threshold and 10 ppm mass accuracy, the MS1 analysis returns 20 true positives and 0 false positives for the example monoclonal antibody dataset.
- All 20 true-positive glycans are identified at the same retention times as the manual analysis.
- Benchmarked threshold sensitivity matches Table 5 expectations: 10 ppm outperforms 50 ppm at 25% intensity (20 TP / 3 FP vs 20 TP / 8 FP); relaxing to 50% intensity drops to 19 TP, and 100% intensity drops to 6 TP / 6 FP.
- MS2 confirmation for a glycan-RT pair (e.g., A1G1F(α1,3) at RT 13.77 min) annotates a plausible fraction of the distinct peaks above 3% relative abundance (18 of 24 in the paper).
- The MS1 output Excel file is produced and lists only glycans present in the supplied MS1 Database (no de novo calls).

## Limitations

- No de novo glycan identification: the app can only identify glycans already present in its databases.
- O-linked glycans are not supported; scope is limited to N-linked glycans.
- The app cannot process multiple mzXML files simultaneously (planned future improvement).
- The isomer peak retention time range is a fixed value, not customizable.
- No software version or changelog was stated for the app in the provided context.

## Evidence

- [results] A relative threshold of 25% and mass error threshold of 10 parts per million (ppm) provided the optimal result for this dataset, containing the fewest false positives while still identifying all 20: "A relative threshold of 25% and mass error threshold of 10 parts per million (ppm) provided the optimal result for this dataset, containing the fewest false positives while still identifying all 20"
- [results] All 20 true positives were identified by the app at the same RTs as with manual analysis.: "All 20 true positives were identified by the app at the same RTs as with manual analysis."
- [intro] ProteoWizard is used to convert the raw MS files to mzXML format.: "ProteoWizard is used to convert the raw MS files to mzXML format."
- [intro] The MS1 output Excel file that the app returns can then be reviewed to determine which glycans are likely to be present in the sample.: "The MS1 output Excel file that the app returns can then be reviewed to determine which glycans are likely to be present in the sample."
- [readme] Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data.: "Welcome to GlyKAn AZ, an application developed to streamline the N-linked glycan identification process from LC-MS/MS data."
