---
name: hsqc-carbon-shift-annotation
description: Use when peak-picked TOCSY and HSQC tables from the same sample are available
  and spin system sub-graphs must be converted into (1H, 13C) features for dereplication
  or similarity networking.
when_to_use_negative:
- Compounds whose 1H signals lie exclusively in the shielded region (e.g., sterols,
  low MW isoprenoids, lipids, volatile terpenes) — TOCSY cross-peaks with both F1
  and F2 < 2.5 ppm are excluded, so these classes cannot be annotated without user-supplied
  reference spectra
- Input is already an annotated spin system feature table — this step would duplicate
  annotation rather than prepare features
- Only 1D 1H spectra are available — the 13C dimension required for annotation comes
  from HSQC cross-peaks
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MADByTE
  role: Platform whose spin system construction module builds directed graphs from
    TOCSY peak tables and annotates nodes with HSQC 13C shifts; exports spin system
    features
  repo: https://github.com/liningtonlab/MADByTE
- name: NMRglue
  role: Supports NMR data handling/visualization of the processed TOCSY and HSQC spectra
    used in the workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:3927cf63da961652dc7d7fa68a8925457413160741b7d3a6339e0674f0912b31
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - MADByTE
  - NMRglue
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hsqc-carbon-shift-annotation

## Summary

Annotates 1H nodes of TOCSY-derived spin system graphs with 13C chemical shifts from matched HSQC peak tables, yielding (1H, 13C) pair features per spin system for downstream MADByTE similarity networking.

## When to use

Use when peak-picked TOCSY and HSQC tables from the same sample are available and spin system sub-graphs must be converted into (1H, 13C) features for dereplication or similarity networking.

## When NOT to use

- Compounds whose 1H signals lie exclusively in the shielded region (e.g., sterols, low MW isoprenoids, lipids, volatile terpenes) — TOCSY cross-peaks with both F1 and F2 < 2.5 ppm are excluded, so these classes cannot be annotated without user-supplied reference spectra
- Input is already an annotated spin system feature table — this step would duplicate annotation rather than prepare features
- Only 1D 1H spectra are available — the 13C dimension required for annotation comes from HSQC cross-peaks

## Inputs

- Peak-picked phase-sensitive 2D TOCSY peak table (symmetrized, f1 corrected against f2, < 2.5 ppm quadrant excluded)
- Peak-picked phase-sensitive 2D HSQC peak table (solvent/water ±0.02 ppm band and illegitimate regions removed)
- Per-sample TOCSY-derived directed spin system graphs (nodes = 1H signals, edges = TOCSY cross-peaks)

## Outputs

- Spin system feature table (e.g., spin_system_features.csv) with (1H, 13C) chemical shift pairs and spin system IDs per sample
- Annotated spin system sub-graphs whose nodes carry 13C shifts from the closest HSQC resonance

## How to apply

First clean the HSQC peak table: remove solvent and water signals in a band of ±0.02 ppm in the 1H dimension, and drop the three illegitimate HSQC regions (1H > 7.0 with 13C < 50 ppm; 1H < 7.0 with 13C > 170 ppm; 1H < 2.4 with 13C > 100 ppm). From the symmetrized TOCSY peak table (f1 shifts corrected against f2, shielded quadrant < 2.5 ppm excluded, TOCSY signals without a corresponding HSQC cross-peak in 1H dropped), build a directed graph per sample where nodes are 1H signals and edges are TOCSY cross-peaks; remove singly-connected nodes and non-reciprocal edges to isolate spin system sub-graphs. Annotate each node with the closest matching 13C chemical shift from the HSQC table; when spectral overlap yields multiple candidate 13C shifts, take the value of the closest HSQC resonance. Export the resulting (1H, 13C) shifts with spin system identifiers for every sample (e.g., spin_system_features.csv) for feature matching and network generation.

## Related tools

- **MADByTE** (Platform whose spin system construction module builds directed graphs from TOCSY peak tables and annotates nodes with HSQC 13C shifts; exports spin system features) — https://github.com/liningtonlab/MADByTE
- **NMRglue** (Supports NMR data handling/visualization of the processed TOCSY and HSQC spectra used in the workflow)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- spin_system_features.csv exists in a valid tabular format for every sample
- The file contains the expected columns: 1H chemical shifts, 13C chemical shifts, and spin system identifiers per sample
- Every exported node carries both a 1H shift and an annotated 13C shift; nodes lacking an HSQC match were excluded upstream
- Annotated 13C values correspond to the closest HSQC resonance in cases of spectral overlap, not an arbitrary candidate
- No retained feature falls in the removed illegitimate HSQC regions (1H > 7.0/13C < 50; 1H < 7.0/13C > 170; 1H < 2.4/13C > 100)

## Limitations

- MADByTE will not identify every spin system in a given mixture
- Spectral overlap can yield multiple candidate 13C chemical shifts; the closest HSQC resonance value is used, which may be ambiguous
- Peak picking performs poorly in the crowded shielded region, and compound classes with exclusively shielded 1H signals (0.4% of predicted natural products) are precluded from analysis unless reference spectra are supplied

## Evidence
