---
name: mass-spec-imaging-pixel-matrix-processing
description: Use when raw, untargeted TOF-SIMS pixel intensities (pixel-by-peak matrices
  exported from scSpaMet notebook 01) need per-pixel total-ion normalization before
  pixel clustering or other pixel-level SIMS analyses.
when_to_use_negative:
- Input is already a normalized pixel matrix (non-zero rows sum to the constant target
  P) or is a single-cell-aggregated metabolite table rather than raw pixel-level SIMS
  intensities
- The task concerns IMC protein preprocessing, cross-modality registration, or VAE
  joint embedding — those are separate scSpaMet steps; this skill covers the SIMS
  metabolite modality only
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: scSpaMet
  role: Its notebook 01 exports the raw SIMS pixel intensities that this skill normalizes;
    the normalized matrix feeds scSpaMet's SIMS pixel clustering
  repo: github.com/coskunlab/ScSpaMet
- name: pandas
  role: Loads the exported SIMS peak-coordinate/intensity files, assembles the pixel-by-peak
    matrix, and writes normalized_pixel_intensities.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43917-5
    title: Single-cell spatial metabolomics with cell-type specific protein profiling
      for tissue systems biology
  source_content_hash: sha256:3930db4d7cd84070aa427bbfd73ffd7ea0148b6927e3502470c5dbb962107b9b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - pandas
  - scSpaMet
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# mass-spec-imaging-pixel-matrix-processing

## Summary

Per-pixel total-ion-count normalization of raw, untargeted TOF-SIMS pixel-by-m/z-peak intensity matrices from the scSpaMet pipeline, applied as a single SIMS-modality preprocessing step before pixel-level downstream analysis such as pixel clustering.

## When to use

Use when raw, untargeted TOF-SIMS pixel intensities (pixel-by-peak matrices exported from scSpaMet notebook 01) need per-pixel total-ion normalization before pixel clustering or other pixel-level SIMS analyses.

## When NOT to use

- Input is already a normalized pixel matrix (non-zero rows sum to the constant target P) or is a single-cell-aggregated metabolite table rather than raw pixel-level SIMS intensities
- The task concerns IMC protein preprocessing, cross-modality registration, or VAE joint embedding — those are separate scSpaMet steps; this skill covers the SIMS metabolite modality only

## Inputs

- Exported SIMS peak-coordinate/intensity files produced by scSpaMet notebook 01 (Processing of IMC (protein) and SIMS (metabolite) images)
- Raw pixel-by-peak TOF-SIMS intensity matrix with pixel identifiers as row keys and m/z peak coordinates as column keys

## Outputs

- normalized_pixel_intensities.csv with pixels as rows and m/z peaks as columns, every non-zero row summing to the constant target total P
- Normalized SIMS pixel-intensity matrix ready for pixel-level downstream analysis such as pixel clustering

## How to apply

Load the SIMS peak-coordinate/intensity files exported by scSpaMet notebook 01 (Processing of IMC (protein) and SIMS (metabolite) images) with pandas and assemble the pixel-by-peak intensity matrix, keeping pixel identifiers as rows and m/z peak coordinates as column keys. Apply the per-pixel total-ion normalization Yi = Xi / (sum_j xij) * P: divide each pixel's spectrum by its summed intensity and scale by the target total P so pixels become comparable regardless of raw total ion counts. Handle zero-total pixels by leaving their rows at zero, avoiding division-by-zero artifacts (NaN/inf). Write the normalized matrix to normalized_pixel_intensities.csv with pixels as rows and m/z peaks as columns; this normalized pixel-intensity data is then used for SIMS pixel clustering, which reveals unique metabolic variation in the spatial context. This is applied as a SIMS-modality-only preprocessing step; IMC protein data follow their own processing path in the pipeline.

## Related tools

- **scSpaMet** (Its notebook 01 exports the raw SIMS pixel intensities that this skill normalizes; the normalized matrix feeds scSpaMet's SIMS pixel clustering) — github.com/coskunlab/ScSpaMet
- **pandas** (Loads the exported SIMS peak-coordinate/intensity files, assembles the pixel-by-peak matrix, and writes normalized_pixel_intensities.csv)

## Examples

```
import pandas as pd; df = pd.read_csv('sims_pixel_intensities.csv'); s = df.sum(axis=1); Y = df.div(s.where(s > 0, 1), axis=0); Y.to_csv('normalized_pixel_intensities.csv', index=False)
```

## Evaluation signals

- Every non-zero row of the output normalized_pixel_intensities.csv sums to the constant P within floating-point tolerance
- Zero-total input pixels remain all-zero rows; the output contains no NaN or inf values
- Output shape matches the expected pixel-by-peak dimensions, with pixel identifiers and m/z peak column keys preserved from the raw export
- Raw pixel totals vary across pixels while normalized rows share a common scale (each sums to P)
- Downstream SIMS pixel clustering on the normalized matrix yields spatially coherent metabolic variation consistent with the paper's reported behavior

## Limitations

- Zero-total pixels must be left at zero during the Yi = Xi / (sum_j xij) * P division to avoid division-by-zero artifacts
- TOF-SIMS imaging regions are relatively small — a trade-off between imaging speed and spatial resolution — limiting pixel yield per field of view
- Untargeted TOF-SIMS peaks are not metabolite-annotated by this step; challenges in metabolite annotation and coverage (lipids, amino acids, metabolic pathways) remain downstream
