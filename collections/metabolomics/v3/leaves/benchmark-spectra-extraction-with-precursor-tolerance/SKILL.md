---
name: benchmark-spectra-extraction-with-precursor-tolerance
description: Use when preparing reference MS/MS spectra from contest or benchmark
  datasets (CASMI 2016/2022) where spectra must be located in raw measurements by
  precursor m/z and retention time before comparing predicted vs. reference spectra.
when_to_use_negative:
- The benchmark data are already organized as per-compound reference spectra in msp/mgf
  library format and need no precursor-matching extraction
- You need spectra from NIST'17 or MS-DIAL library splits, which are processed from
  .msp libraries rather than extracted from raw challenge measurements
- Collision energies are missing or in unclear formats, which the article explicitly
  excluded from processing
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: FIORA
  role: Consumes the extracted benchmark query CSVs (Name, SMILES, Precursor_type,
    CE, Instrument_type) to predict spectra via fiora-predict and evaluate them against
    extracted reference spectra with fiora-eval
  repo: https://github.com/BAMeScience/fiora
- name: OpenMS
  role: Used to extract CASMI 2022 challenge spectra from raw measurements using a
    10 ppm precursor tolerance and 5-second retention time window
- name: CFM-ID
  role: Baseline predictor run on the same extracted benchmark queries; predictions
    evaluated at the closest matching collision energy among 10, 20, 40 eV
- name: ICEBERG
  role: Retrained baseline predictor (from the ms-pred repository, test compounds
    excluded from training) run on the same extracted test splits
  repo: https://github.com/samgoldman97/ms-pred
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:cbcda84f73529057911f89b1416cbc356d27600dc8e0d2f174ecdf96e28d3b45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - CFM-ID
  - FIORA
  - ICEBERG
  - OpenMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Benchmark spectra extraction with precursor tolerance

## Summary

Extracts reference challenge spectra from raw benchmark contest data (e.g., CASMI 2022) by matching precursor masses within a 10 ppm tolerance and retention times within a 5-second window, then merges replicate measurements at the same collision energy into consensus spectra for downstream spectral-similarity evaluation.

## When to use

Use when preparing reference MS/MS spectra from contest or benchmark datasets (CASMI 2016/2022) where spectra must be located in raw measurements by precursor m/z and retention time before comparing predicted vs. reference spectra.

## When NOT to use

- The benchmark data are already organized as per-compound reference spectra in msp/mgf library format and need no precursor-matching extraction
- You need spectra from NIST'17 or MS-DIAL library splits, which are processed from .msp libraries rather than extracted from raw challenge measurements
- Collision energies are missing or in unclear formats, which the article explicitly excluded from processing

## Inputs

- CASMI 2022 challenge raw measurements with precursor m/z and retention time metadata
- CASMI 2016 challenge data acquired with stepped collision energies
- Precursor type annotations ([M+H]+, [M-H]-; [M]+/[M]- for MSnLib)
- Collision energy and instrument type metadata per challenge entry

## Outputs

- Extracted reference challenge spectra matched at 10 ppm precursor tolerance and 5 s RT window
- Consensus spectra per collision energy for CASMI 2022
- Benchmark query CSVs with columns Name, SMILES, Precursor_type, CE, Instrument_type for fiora-predict

## How to apply

Extract challenge spectra from the benchmark raw data using a precursor tolerance of 10 ppm and a retention time window of 5 seconds. For CASMI 2022, merge multiple measurements acquired at the same collision energy into a consensus spectrum; handle missing collision energies explicitly, since not all energies are found for all challenge spectra. For CASMI 2016, note that a stepped collision energy was used, so predictions across the 3 collision energies must later be merged into a single 'stepped' spectrum rather than the references. Filter extraction to [M+H]+ and [M-H]- precursor modes (plus [M]+/[M]- for MSnLib) so the resulting query tables (Name, SMILES, Precursor_type, CE, Instrument_type) feed directly into fiora-predict and comparison with CFM-ID/ICEBERG predictions.

## Related tools

- **FIORA** (Consumes the extracted benchmark query CSVs (Name, SMILES, Precursor_type, CE, Instrument_type) to predict spectra via fiora-predict and evaluate them against extracted reference spectra with fiora-eval) — https://github.com/BAMeScience/fiora
- **OpenMS** (Used to extract CASMI 2022 challenge spectra from raw measurements using a 10 ppm precursor tolerance and 5-second retention time window)
- **CFM-ID** (Baseline predictor run on the same extracted benchmark queries; predictions evaluated at the closest matching collision energy among 10, 20, 40 eV)
- **ICEBERG** (Retrained baseline predictor (from the ms-pred repository, test compounds excluded from training) run on the same extracted test splits) — https://github.com/samgoldman97/ms-pred

## Evaluation signals

- Each extracted CASMI 2022 spectrum is matched within 10 ppm precursor tolerance and a 5-second retention time window
- Replicate measurements at the same collision energy are reduced to exactly one consensus spectrum per compound/energy pair
- Extracted query tables carry the required columns Name, SMILES, Precursor_type, CE, Instrument_type, restricted to [M+H]+ and [M-H]- (plus [M]+/[M]- for MSnLib)
- Missing or unclear-format collision-energy spectra are excluded rather than silently kept
- Downstream fiora-eval produces per-split result CSVs (e.g., test_eval.csv) whose median cosine similarities can be aggregated into the Table 1 comparison

## Limitations

- Despite extraction efforts, not all collision energies for all CASMI 22 challenge spectra were found
- MS/MS spectra with missing collision energies or unclear formats were excluded
- CFM-ID was trained on the METLIN17 library, so train/test compound separation is not guaranteed for CFM-ID comparisons

## Evidence

- [discussion] Challenge spectra were extracted using a precursor tolerance of 10 ppm and retention time window of 5 seconds.: "Challenge spectra were extracted using a precursor tolerance of 10 ppm and retention time window of 5 seconds."
- [discussion] Multiple measurements at the same collision energy were merged into a consensus spectrum.: "Multiple measurements at the same collison energy were merged into a consensus spectrum."
- [discussion] In the CASMI 16 challenge, a stepped collision energy was used, so we predicted spectra with FIORA for all 3 collision energies and merged them.: "In the CASMI 16 challenge, a stepped collision energy was used, so we predicted spectra with FIORA for all 3 collision energies and merged them into a single “stepped" spectrum."
- [discussion] we selected the CASMI 2016 and CASMI 2022 datasets for benchmarking. Both datasets were downloaded from the CASMI contest webpage: "we selected the CASMI 2016 and CASMI 2022 datasets for benchmarking. Both datasets were downloaded from the CASMI contest webpage"
- [discussion] MS/MS spectra with missing collision energies or unclear formats were excluded.: "MS/MS spectra with missing collision energies or unclear formats were excluded."
