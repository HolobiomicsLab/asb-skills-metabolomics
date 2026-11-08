---
name: crosspeak-correlation-graph-construction
description: Use when peak-picked TOCSY and HSQC peak lists from complex NMR mixtures
  must be converted into per-sample spin system features (paired 1H/13C resonances)
  for downstream feature matching or molecular networking.
when_to_use_negative:
- The sample set contains compound classes with exclusively shielded 1H signals (e.g.
  sterols, low MW isoprenoids, lipids, volatile terpenes) — the < 2.5 ppm TOCSY quadrant
  is excluded so these spin systems cannot be constructed.
- Inputs are raw FIDs or unprocessed spectra without peak-picked TOCSY/HSQC tables
  — peak picking and preprocessing (solvent filtering, symmetrization) must precede
  graph construction.
- The goal is dereplication of compounds lacking user-supplied reference spectra —
  MADByTE only dereplicates compound classes when reference spectra are provided.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MADByTE
  role: 'Platform that executes the spin system construction: directed graph from
    TOCSY peak table, pruning, HSQC 13C annotation, and feature export'
  repo: https://github.com/liningtonlab/MADByTE
- name: NMRglue
  role: Supports NMR spectrum handling/visualization underlying the peak-table inputs
    to graph construction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:bc53a11333fc95f25b73a751270f0b61d35a2107fc36f1eac5e7d9ac48cd790c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - MADByTE
  - NMRglue
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# crosspeak-correlation-graph-construction

## Summary

Constructs spin system features from paired 2D TOCSY/HSQC peak tables by building a directed graph per sample (nodes = 1H TOCSY signals, edges = TOCSY cross-peaks), pruning singly-connected nodes and non-reciprocal edges, and annotating each node with the closest matching 13C shift from the HSQC peak table.

## When to use

Use when peak-picked TOCSY and HSQC peak lists from complex NMR mixtures must be converted into per-sample spin system features (paired 1H/13C resonances) for downstream feature matching or molecular networking.

## When NOT to use

- The sample set contains compound classes with exclusively shielded 1H signals (e.g. sterols, low MW isoprenoids, lipids, volatile terpenes) — the < 2.5 ppm TOCSY quadrant is excluded so these spin systems cannot be constructed.
- Inputs are raw FIDs or unprocessed spectra without peak-picked TOCSY/HSQC tables — peak picking and preprocessing (solvent filtering, symmetrization) must precede graph construction.
- The goal is dereplication of compounds lacking user-supplied reference spectra — MADByTE only dereplicates compound classes when reference spectra are provided.

## Inputs

- Peak-picked 2D TOCSY peak table (phase-sensitive, symmetrized, f1 corrected against f2)
- Peak-picked 2D HSQC peak table (phase-sensitive, solvent/water and illegitimate-region filtered)
- NMR solvent identity (selected for solvent/water band filtering)

## Outputs

- Per-sample spin system feature table (paired 1H and 13C chemical shifts with spin system identifiers), e.g. spin_system_features.csv

## How to apply

First preprocess the peak tables: filter HSQC peaks to remove solvent/water signals (±0.02 ppm band in the 1H dimension) and the three illegitimate HSQC regions (1H > 7.0 with 13C < 50 ppm; 1H < 7.0 with 13C > 170 ppm; 1H < 2.4 with 13C > 100 ppm), then exclude the shielded TOCSY quadrant (< 2.5 ppm), symmetrize TOCSY f1 shifts against f2, and drop TOCSY signals lacking a corresponding HSQC cross-peak in the 1H dimension. Next, for each sample, generate a directed graph from the TOCSY peak table where nodes represent 1H signals and edges represent TOCSY cross-peaks; remove nodes containing only a single connection to a spin system and remove edges that are not reciprocal, leaving one sub-graph per unique spin system. Annotate each node with the closest matching 13C chemical shift from the filtered HSQC peak table (using the closest HSQC resonance when spectral overlap yields multiple candidates). Export the resulting (1H, 13C) pairs with spin system identifiers per sample, e.g. to spin_system_features.csv, using the MADByTE platform (NMRglue-backed).

## Related tools

- **MADByTE** (Platform that executes the spin system construction: directed graph from TOCSY peak table, pruning, HSQC 13C annotation, and feature export) — https://github.com/liningtonlab/MADByTE
- **NMRglue** (Supports NMR spectrum handling/visualization underlying the peak-table inputs to graph construction)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- spin_system_features.csv exists in valid tabular format for every sample
- The file contains the expected spin system feature columns: 1H chemical shift, 13C chemical shift, and spin system IDs per sample
- Each exported node carries a paired (1H, 13C) value — nodes without a matching HSQC cross-peak in the 1H dimension must not appear
- No exported feature falls in the excluded regions: TOCSY cross-peaks with both F1 and F2 < 2.5 ppm, or the three illegitimate HSQC regions (e.g. 1H > 7.0 with 13C < 50 ppm)
- Every spin system sub-graph has at least two connected nodes (single-connection nodes removed) and all edges are reciprocal

## Limitations

- MADByTE will not identify every spin system in a given mixture.
- TOCSY cross-peaks where the chemical shifts in F1 and F2 are both < 2.5 ppm are excluded, precluding analysis of specialist compound classes (sterols, low MW isoprenoids, lipids, volatile terpenes) with exclusively shielded 1H signals.
- Peak-picking algorithms perform poorly in the crowded shielded region.
- Where spectral overlap yields multiple candidate 13C chemical shifts, only the closest HSQC resonance value is assigned, which may misannotate overlapped nodes.

## Evidence

- [other] validation of outputs: [evidence span withheld — anchor status: not_found]
