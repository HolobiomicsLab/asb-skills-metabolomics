---
name: tocsy-symmetrization
description: Use when processing peak-picked TOCSY/HSQC tables from complex mixture
  NMR (e.g. MADByTE) and f1/f2 chemical shifts are asymmetric, so TOCSY data must
  be symmetrized and filtered before spin system construction.
when_to_use_negative:
- The input is already a spin system feature table or similarity network — symmetrization
  operates on raw TOCSY peak lists only
- The sample's compounds contain exclusively shielded 1H signals (e.g. sterols, low
  MW isoprenoids, lipids, volatile terpenes) — the <2.5 ppm quadrant exclusion precludes
  their analysis
- Absolute-value HSQC data is being used without proper phase-sensitive acquisition,
  since this workflow was designed around phase-sensitive HSQC peak lists
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MADByTE
  role: Runs the TOCSY symmetrization and filtering as part of its peak-picking-to-spin-system
    workflow; reads the TOCSY and HSQC peak tables and produces spin system features
  repo: https://github.com/liningtonlab/MADByTE
- name: NMRglue
  role: Used by MADByTE for spectrum visualization of the 1D/2D data during processing
    and quality checks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:7cec9072f67c7862192eead9f1a02fe34c066481671777b738c1e74aebd5184c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - MADByTE
  - NMRglue
  edam_operation: http://edamontology.org/operation_3216
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

# tocsy-symmetrization

## Summary

Symmetrize phase-sensitive 2D TOCSY spectra by correcting f1 chemical shifts against the corresponding f2 values, then exclude the shielded quadrant (<2.5 ppm) and TOCSY signals lacking HSQC counterparts, before building spin system graphs in MADByTE.

## When to use

Use when processing peak-picked TOCSY/HSQC tables from complex mixture NMR (e.g. MADByTE) and f1/f2 chemical shifts are asymmetric, so TOCSY data must be symmetrized and filtered before spin system construction.

## When NOT to use

- The input is already a spin system feature table or similarity network — symmetrization operates on raw TOCSY peak lists only
- The sample's compounds contain exclusively shielded 1H signals (e.g. sterols, low MW isoprenoids, lipids, volatile terpenes) — the <2.5 ppm quadrant exclusion precludes their analysis
- Absolute-value HSQC data is being used without proper phase-sensitive acquisition, since this workflow was designed around phase-sensitive HSQC peak lists

## Inputs

- Processed phase-sensitive 2D TOCSY peak-picked table (from Zenodo deposit https://doi.org/10.5281/zenodo.4317721 or user-acquired data)
- Filtered HSQC peak-picked table (solvent/water regions removed at ±0.02 ppm in 1H; three illegitimate regions excluded)

## Outputs

- Symmetrized, filtered TOCSY peak table ready for directed-graph spin system construction
- TOCSY 1H signals retained only where an HSQC cross-peak exists in the 1H dimension

## How to apply

First exclude the shielded quadrant of the TOCSY spectrum (<2.5 ppm); note that cross-peaks where shifts in BOTH F1 and F2 are <2.5 ppm are removed. Symmetrize the TOCSY data by correcting f1 chemical shifts against the corresponding f2 values. Then exclude TOCSY signals that lack a corresponding HSQC cross-peak in the 1H dimension. This conditioning produces a clean directed-graph input where nodes represent 1H signals and edges are TOCSY cross-peaks, which MADByTE prunes by removing singly-connected nodes and non-reciprocal edges before annotating nodes with 13C shifts from the HSQC table.

## Related tools

- **MADByTE** (Runs the TOCSY symmetrization and filtering as part of its peak-picking-to-spin-system workflow; reads the TOCSY and HSQC peak tables and produces spin system features) — https://github.com/liningtonlab/MADByTE
- **NMRglue** (Used by MADByTE for spectrum visualization of the 1D/2D data during processing and quality checks)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- TOCSY table is asymmetric before symmetrization (f1 vs f2 mismatch) and symmetric after — spot-check a few cross-peaks for matching f1/f2 values
- No retained TOCSY cross-peak has both F1 and F2 chemical shifts < 2.5 ppm
- Every retained TOCSY 1H signal has a corresponding HSQC cross-peak in the 1H dimension
- Downstream spin_system_features.csv is produced in valid tabular format with 1H/13C shifts and spin system IDs for each sample
- Solvent and water 1H signals (±0.02 ppm band) are absent from the HSQC-filtered companion table

## Limitations

- MADByTE will not identify every spin system in a given mixture
- Peak-picking algorithms perform poorly in the crowded shielded region, motivating the <2.5 ppm exclusion
- Compound classes with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes) are precluded from analysis unless the user supplies reference spectra

## Evidence
