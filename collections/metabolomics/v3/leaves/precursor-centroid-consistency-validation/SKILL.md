---
name: precursor-centroid-consistency-validation
description: Use when processing large batches of raw LC–MS/MS spectra (e.g. mined
  from MassIVE GNPS .mzML experiments) where fragment centroids must be validated
  against precursor m/z prior to ML training or spectrum library curation.
when_to_use_negative:
- When spectra have already passed library-level QC (e.g. MoNA or NIST20 reference
  entries) — this skill is designed for unannotated, repository-scale mined spectra,
  not curated libraries.
- When the goal is data augmentation or training-data generation rather than quality
  control — re-running Algorithm 2 on already-filtered GeMS subsets would only shrink
  the training corpus without adding information.
- When analyzing MS1 isotopic patterns or adduct distributions — the DreaMS pipeline
  explicitly disregards these and focuses solely on tandem mass spectra.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: HDF5
  role: Storage format for the mined spectra and their metadata across the GeMS-A/B/C
    filtering stages
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:b15a527d2fd25153bb9ca3de55d1ffca139e170c3af8f10661394e51fb70f75b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HDF5
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# precursor-centroid-consistency-validation

## Summary

Spectrum-level QC step from the GeMS mining pipeline that checks whether fragment peaks are internally consistent with the precursor m/z (peak count, precursor charge, m/z range, centroid-vs-precursor agreement) before a mined MassIVE GNPS spectrum is admitted to the GeMS-A/B/C subsets.

## When to use

Use when processing large batches of raw LC–MS/MS spectra (e.g. mined from MassIVE GNPS .mzML experiments) where fragment centroids must be validated against precursor m/z prior to ML training or spectrum library curation.

## When NOT to use

- When spectra have already passed library-level QC (e.g. MoNA or NIST20 reference entries) — this skill is designed for unannotated, repository-scale mined spectra, not curated libraries.
- When the goal is data augmentation or training-data generation rather than quality control — re-running Algorithm 2 on already-filtered GeMS subsets would only shrink the training corpus without adding information.
- When analyzing MS1 isotopic patterns or adduct distributions — the DreaMS pipeline explicitly disregards these and focuses solely on tandem mass spectra.

## Inputs

- HDF5-based GeMS-format collection of spectra mined from ~250,000 LC–MS/MS experiments in MassIVE GNPS (GNPS Experimental Mass Spectra)
- Per-spectrum records with fragment centroid m/z values, precursor m/z and precursor charge, plus LC–MS/MS metadata (instrument type)
- Output of upstream file-level QC (Algorithm 1 instrument m/z accuracy filter via XICs; retention-time and MS-level ordering checks)

## Outputs

- Flagged/filtered spectra partitioned into nested GeMS-A (42 M), GeMS-B (100 M) and GeMS-C (201 M) subsets with consecutively larger size at the expense of quality
- Per-spectrum QC decision (accept/reject) from Algorithm 2 with reasons such as inconsistent centroids vs precursor m/z
- HDF5 storage of accepted spectra with selected LC–MS/MS metadata including instrument type

## How to apply

Load the mined collection of LC–MS/MS experiments from the MassIVE GNPS repository stored in the compact HDF5-based GeMS format, after file-level checks (retention-time and MS-level ordering, Algorithm 1 LC–MS instrument accuracy via XICs) have already been applied. For each spectrum run Algorithm 2: inspect the peak count, verify the precursor charge state, confirm the m/z range falls within instrument limits, and check consistency of fragment centroids with the precursor m/z — spectra whose centroids imply fragments heavier than their precursor are rejected. Apply these checks incrementally to build the three nested subsets: GeMS-A (basic criteria, 42 M spectra), GeMS-B (adds instrument m/z accuracy, 100 M spectra) and GeMS-C (adds number of high-intensity signals, 201 M spectra), where quality is traded for size. Store each accepted spectrum together with selected LC–MS/MS metadata such as instrument type in the HDF5-based binary format; retain the instrument-composition profile per subset (e.g. GeMS-A is 97% Orbitrap, GeMS-C is 52% Orbitrap / 41% QTOF) as an audit artifact.

## Related tools

- **HDF5** (Storage format for the mined spectra and their metadata across the GeMS-A/B/C filtering stages)

## Examples

```
python scripts/gems_qc.py --input gems_a.hdf5 --algorithm 2 --check precursor-consistency --output gems_filtered.hdf5
```

## Evaluation signals

- Resulting subset sizes match the reported GeMS-A/B/C counts of approximately 42, 100 and 201 million spectra respectively
- Instrument-type composition per subset matches the published profile: GeMS-A ~97% Orbitrap; GeMS-C ~52% Orbitrap and ~41% QTOF
- No accepted spectrum contains fragment centroids whose m/z values are inconsistent with (exceed) the precursor m/z given the recorded precursor charge
- Every stored spectrum carries its selected LC–MS/MS metadata (instrument type) in the HDF5 file
- Rejected-spectrum logs show the checks actually discriminating — peak count, precursor charge, m/z range and centroid-consistency reasons all occur

## Limitations

- The quality/size trade-off is inherent: GeMS-A is highest quality (97% Orbitrap) but smallest, while GeMS-C (201 M spectra, 52% Orbitrap / 41% QTOF) trades quality for coverage; choose the subset deliberately per task.
- Pre-training in the paper used only a subset of available mass spectra; running the filter is not sufficient to unlock full performance on raw unannotated spectra.
- The mining pipeline focuses on tandem mass spectra only and disregards MS1 isotopic patterns and adduct distributions, so centroid-consistency validation will not catch issues in those features.

## Evidence

- [methods] Quality criteria (e.g. instrument m/z accuracy per LC–MS/MS experiment, number of high-intensity signals) yield three subsets of consecutively larger size at the expense of quality: [evidence span withheld — anchor status: not_found]
- [methods] Apply Algorithm 2 at the spectrum level: [evidence span withheld — anchor status: not_found]
- [methods] Incrementally filter the spectra into three nested subsets: [evidence span withheld — anchor status: not_found]
- [abstract] HDF5-based binary format: "we stored the GeMS spectra, including selected LC–MS/MS metadata, in our compact HDF5-based binary format designed for deep learning"
- [abstract] quality control algorithms filter into three subsets: "we developed a pipeline of quality control algorithms allowing us to filter the collected spectra into three subsets—GeMS-A, GeMS-B and GeMS-C"
- [discussion] disregards MS1 isotopic patterns and adduct distributions: "our method is focused solely on tandem mass spectra, disregarding other important features such as MS1 isotopic patterns or adduct distributions"
