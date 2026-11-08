---
name: collision-energy-matching-and-stepped-spectrum-merging
description: Use when evaluating simulated MS/MS spectra against challenge or reference
  sets whose acquisition used stepped collision energies or multiple CE replicate
  measurements, so predicted and experimental spectra must be compared on a consistent
  collision-energy basis.
when_to_use_negative:
- Reference and predicted spectra were both acquired at a single, directly matching
  collision energy — then a one-to-one comparison without CE merging or matching is
  appropriate
- The task is training or fine-tuning a spectral predictor rather than benchmarking
  prediction quality against reference challenge spectra
- Compound queries lack collision-energy annotations, since spectra with missing collision
  energies or unclear formats were excluded from this benchmark
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FIORA
  role: Predicts MS/MS spectra at each of the 3 stepped collision energies that are
    subsequently merged into a single stepped spectrum for CASMI 16
  repo: https://github.com/BAMeScience/fiora
- name: CFM-ID
  role: Supplies pre-computed predictions at 10, 20, and 40 eV from which the prediction
    at the closest matching collision energy is selected for comparison
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-57422-4
    title: 'FIORA: Local neighborhood-based prediction of compound mass spectra from
      single fragmentation events'
  source_content_hash: sha256:e2fa1864d6076d90ff669cc6485eb0ccd1bea2eb94ebaa68c924e439a90e117d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BAMeScience/fiora
  tools:
  - CFM-ID
  - FIORA
  - ICEBERG
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# collision-energy-matching-and-stepped-spectrum-merging

## Summary

A benchmark-evaluation procedure for aligning predicted and reference tandem MS/MS spectra acquired at mismatched collision energies: merging multi-CE predictions into single 'stepped' spectra (CASMI 16), consolidating replicate measurements into consensus spectra (CASMI 22), and selecting CFM-ID predictions at the closest matching collision energy among 10, 20, and 40 eV.

## When to use

Use when evaluating simulated MS/MS spectra against challenge or reference sets whose acquisition used stepped collision energies or multiple CE replicate measurements, so predicted and experimental spectra must be compared on a consistent collision-energy basis.

## When NOT to use

- Reference and predicted spectra were both acquired at a single, directly matching collision energy — then a one-to-one comparison without CE merging or matching is appropriate
- The task is training or fine-tuning a spectral predictor rather than benchmarking prediction quality against reference challenge spectra
- Compound queries lack collision-energy annotations, since spectra with missing collision energies or unclear formats were excluded from this benchmark

## Inputs

- Benchmark query CSVs with columns Name, SMILES, Precursor_type, CE, Instrument_type (e.g., CASMI 2016/2022 compound lists filtered to [M+H]+ and [M-H]-)
- FIORA-predicted spectra at 3 stepped collision energies (mgf/msp format) for CASMI 16
- CFM-ID v4 predictions at 10, 20, and 40 eV per compound
- Raw CASMI 2022 challenge spectra with precursor m/z and retention time metadata for OpenMS extraction
- Replicate CASMI 2022 measurements grouped by collision energy

## Outputs

- Single merged 'stepped' FIORA spectra (one per CASMI 16 compound, spanning all 3 CEs)
- Consensus CASMI 2022 spectra (replicates merged per collision energy)
- Per-compound CFM-ID prediction selected at the closest matching collision energy
- Cosine similarity values (FIORA spectral_sqrt_cosine on square-root-transformed intensities, precursor peak included) feeding per-split result CSVs such as test_eval.csv

## How to apply

First determine the collision-energy structure of the target set: CASMI 2016 used a stepped collision energy, so predict spectra with FIORA (via fiora-predict, with Name, SMILES, Precursor_type, CE, Instrument_type query CSVs) at all 3 collision energies and merge them into a single 'stepped' spectrum. For CASMI 2022, extract challenge spectra with OpenMS using a 10 ppm precursor tolerance and a 5 second retention time window, then merge multiple measurements taken at the same collision energy into one consensus spectrum. When comparing against CFM-ID v4 (run in the wishartlab/cfmid Docker container), pick the prediction with the closest matching collision energy among the pre-computed 10, 20, and 40 eV predictions, following the approach of Wang et al. The rationale is that the spectral_sqrt_cosine metric on square-root-transformed intensities is only meaningful when predicted and reference spectra span comparable fragmentation regimes, so CE mismatch would otherwise bias similarity scores.

## Related tools

- **FIORA** (Predicts MS/MS spectra at each of the 3 stepped collision energies that are subsequently merged into a single stepped spectrum for CASMI 16) — https://github.com/BAMeScience/fiora
- **CFM-ID** (Supplies pre-computed predictions at 10, 20, and 40 eV from which the prediction at the closest matching collision energy is selected for comparison) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
fiora-predict -i casmi16_input.csv -o casmi16_specs.mgf
```

## Evaluation signals

- Every CASMI 16 compound has exactly one merged FIORA spectrum covering all 3 stepped collision energies, not three separate per-CE spectra
- CASMI 2022 extracted spectra satisfy the 10 ppm precursor tolerance and 5 second retention time window; replicates at the same collision energy are collapsed to one consensus spectrum
- Each CFM-ID comparison uses the prediction whose collision energy is the closest match among {10, 20, 40} eV to the reference acquisition energy
- Result CSVs (e.g., test_eval.csv) written via fiora-eval --output-dir contain cosine similarities computed on square-root-transformed intensities including the precursor peak
- Aggregated medians per model, test set, and ionization mode reproduce the ordering in Table 1 (FIORA highest on test split, MSnLib, and CASMI 16, outperforming the runner-up by 10% to 49%)

## Limitations

- Not all collision energies for all CASMI 2022 challenge spectra were found during extraction
- CFM-ID was evaluated with its pre-trained METLIN17 model because retraining CFM-ID with the benchmark dataset was computationally infeasible, so train/test compound separation is not guaranteed for CFM-ID
- Consensus merging and stepped merging introduce spectrum-level aggregation choices that differ across models and may affect comparability

## Evidence

- [discussion] CASMI 16 stepped-CE merging: "In the CASMI 16 challenge, a stepped collision energy was used, so we predicted spectra with FIORA for all 3 collision energies and merged them into a single"
- [discussion] CFM-ID closest-CE selection: "Therefore, we evaluated the prediction with the closest matching collision energy, following the approach of Wang et al.23."
- [discussion] CASMI 22 extraction parameters: "Challenge spectra were extracted using a precursor tolerance of 10 ppm and retention time window of 5 seconds."
- [discussion] Consensus merging of replicates: "Multiple measurements at the same collison energy were merged into a consensus spectrum."
- [readme] fiora-predict CLI supports per-query CE specification: "Input files are expected to be in csv format. With a header defining the columns: "Name", "SMILES", "Precursor_type", "CE", "Instrument_type""
