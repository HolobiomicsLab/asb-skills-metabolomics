---
name: scms-noise-background-subtraction
description: Use when raw single-cell peak lists exported from Thermo Xcalibur must
  be denoised, debackgrounded, and TIC-normalized before peak alignment or PCA/machine-learning
  analysis of SCMS data.
when_to_use_negative:
- Input is already a pretreated, aligned peak-intensity matrix (e.g. the dataset.csv
  used by MetaPhenotype) — reapplying subtraction would remove valid signals.
- Data are bulk metabolomics measurements rather than single-probe SCMS single-cell
  peak lists; the 3×10^3 relative-intensity noise cutoff is calibrated for single-cell
  spectra.
- Goal is peak alignment itself (Geena2, 10 ppm, 150–1500 m/z) rather than upstream
  noise/background removal.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: Thermo Xcalibur Qual Browser 3.0
  role: Source of the raw per-cell peak lists (m/z, retention time, intensity) that
    undergo noise and background subtraction and TIC normalization.
- name: Geena2
  role: 'Downstream consumer: the TIC-normalized, debackgrounded matrix is aligned
    by Geena2 (10 ppm tolerance, 150–1500 m/z) after this pretreatment step.'
- name: R
  role: Hosts the in-house background-subtraction script used to remove background
    species from the raw peak lists.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.4c02038
    title: 'MetaPhenotype: A Transferable Meta-Learning Model for Single-Cell Mass
      Spectrometry-Based Cell Phenotype Prediction Using Limited Number of Cells'
  source_content_hash: sha256:523300234557782c1738fdd46d2a800d575b44dacdb13fbee2865135bde6bae0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Geena2
  - R
  - Thermo Xcalibur Qual Browser 3.0
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# scms-noise-background-subtraction

## Summary

Pretreat raw Single-probe SCMS peak lists by removing low-intensity noise species (relative intensity < 3×10^3), subtracting background species with an in-house R script, and TIC-normalizing each cell's intensities before downstream phenotype-classification modeling.

## When to use

Use when raw single-cell peak lists exported from Thermo Xcalibur must be denoised, debackgrounded, and TIC-normalized before peak alignment or PCA/machine-learning analysis of SCMS data.

## When NOT to use

- Input is already a pretreated, aligned peak-intensity matrix (e.g. the dataset.csv used by MetaPhenotype) — reapplying subtraction would remove valid signals.
- Data are bulk metabolomics measurements rather than single-probe SCMS single-cell peak lists; the 3×10^3 relative-intensity noise cutoff is calibrated for single-cell spectra.
- Goal is peak alignment itself (Geena2, 10 ppm, 150–1500 m/z) rather than upstream noise/background removal.

## Inputs

- Raw single-cell peak lists (m/z, retention time, intensity) exported from Thermo Xcalibur Qual Browser 3.0 for each analyzed cell
- Previously established in-house R background-subtraction script

## Outputs

- Noise- and background-subtracted, TIC-normalized per-cell peak-intensity matrix ready for Geena2 peak alignment

## How to apply

Export each cell's peak list (m/z, retention time, intensity) from Thermo Xcalibur Qual Browser 3.0, then subtract noise by discarding species with relative intensity below 3×10^3. Remove background species using the previously established in-house R background-subtraction script. Normalize each cell's peak intensities by total ion current (TIC) so cells are comparable across injections; the resulting matrix is then ready for Geena2 peak alignment (10 ppm tolerance, 150–1500 m/z) and removal of rows with missing values in >50% of cells.

## Related tools

- **Thermo Xcalibur Qual Browser 3.0** (Source of the raw per-cell peak lists (m/z, retention time, intensity) that undergo noise and background subtraction and TIC normalization.)
- **Geena2** (Downstream consumer: the TIC-normalized, debackgrounded matrix is aligned by Geena2 (10 ppm tolerance, 150–1500 m/z) after this pretreatment step.)
- **R** (Hosts the in-house background-subtraction script used to remove background species from the raw peak lists.)

## Examples

```
Rscript background_subtraction.R --input xcalibur_peaks.csv --output tic_normalized.csv
```

## Evaluation signals

- No species with relative intensity below 3×10^3 remains in the per-cell peak list after noise subtraction.
- Each cell's peak intensities sum to the total ion current (TIC) — verify TIC normalization was applied per cell.
- Background species flagged by the in-house R script are absent from the output matrix.
- The pretreated matrix retains the per-cell peak structure (one row-set per analyzed cell) expected by Geena2 alignment.
- Rows are only removed for >50% missing values during the downstream step, not during this subtraction/normalization step — confirming the steps are ordered correctly.

## Limitations

- The 3×10^3 relative-intensity noise cutoff and the R background script are instrument- and protocol-specific (Thermo LTQ Orbitrap XL Single-probe SCMS); they may not transfer to other mass spectrometers or acquisition modes.
- TIC normalization assumes comparable total signal per cell; unusually low-intensity cells may be over-amplified.
- The article describes this as a 'previously established protocol' — the in-house R script itself is not provided in the README or article text.
- This step is upstream of Geena2 alignment; species removed here cannot be recovered by later alignment.

## Evidence

- [other] finding: SCMS pretreatment exports peaks with Xcalibur, performs noise subtraction (relative intensity <3×10^3), background subtraction via an in-house R script, TIC normalization: [evidence span withheld — anchor status: not_found]
