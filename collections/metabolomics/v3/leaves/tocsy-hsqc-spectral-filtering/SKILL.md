---
name: tocsy-hsqc-spectral-filtering
description: Use when processing peak-picked TOCSY/HSQC peak tables from complex natural
  product mixtures prior to spin system construction and filtering out solvent, water,
  and illegitimate-region artifacts is needed.
when_to_use_negative:
- The target compound classes (e.g. sterols, low molecular weight isoprenoids, lipids,
  volatile terpenes) have exclusively shielded 1H signals — the < 2.5 ppm TOCSY quadrant
  exclusion precludes their analysis unless the user supplies reference spectra outside
  this pipeline's scope
- Input is already a cleaned, spin-system feature table (e.g. spin_system_features.csv)
  rather than raw peak-picked TOCSY/HSQC tables
- The data are 1D or heteronuclear experiments other than paired phase-sensitive TOCSY/HSQC,
  since filters rely on the TOCSY/HSQC 1H correspondence
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: MADByTE
  role: Implements the filter cascade (solvent/water ±0.02 ppm band, three illegitimate
    HSQC regions, TOCSY < 2.5 ppm shielded quadrant, f1-vs-f2 symmetrization, HSQC
    cross-peak matching) as part of spin_system_construction
  repo: https://github.com/liningtonlab/MADByTE
- name: NMRglue
  role: Supports direct visualization of 1D spectra in the MADByTE panels so users
    can check peak picking and filter behavior on the underlying spectra
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_006
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:736e7f06435fb63e8d94dd0df771b1b036d2c8c30163463ea4fd4b3e18bba740
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  - NMRglue
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tocsy-hsqc-spectral-filtering

## Summary

Preprocessing filter cascade that cleans peak-picked 2D TOCSY and HSQC tables (solvent/water bands, illegitimate HSQC regions, TOCSY shielded quadrant, orphaned TOCSY signals) before spin system graph construction in the MADByTE platform.

## When to use

Use when processing peak-picked TOCSY/HSQC peak tables from complex natural product mixtures prior to spin system construction and filtering out solvent, water, and illegitimate-region artifacts is needed.

## When NOT to use

- The target compound classes (e.g. sterols, low molecular weight isoprenoids, lipids, volatile terpenes) have exclusively shielded 1H signals — the < 2.5 ppm TOCSY quadrant exclusion precludes their analysis unless the user supplies reference spectra outside this pipeline's scope
- Input is already a cleaned, spin-system feature table (e.g. spin_system_features.csv) rather than raw peak-picked TOCSY/HSQC tables
- The data are 1D or heteronuclear experiments other than paired phase-sensitive TOCSY/HSQC, since filters rely on the TOCSY/HSQC 1H correspondence

## Inputs

- Peak-picked 2D TOCSY peak table (1H x 1H cross-peaks, f1/f2 chemical shifts)
- Peak-picked phase-sensitive 2D HSQC peak table (1H/13C chemical shifts)
- NMR solvent identity (e.g. DMSO-d6) selected in the MADByTE GUI

## Outputs

- Filtered TOCSY peak list (shielded quadrant and non-HSQC-matched signals removed, symmetrized f1 shifts)
- Filtered HSQC peak list (solvent/water band and three illegitimate regions removed)
- Cleaned peak tables ready for directed-graph spin system construction and 13C annotation

## How to apply

Run the filters on the peak-picked tables inside MADByTE's spin_system_construction step, in order: (1) remove HSQC signals in the solvent and water peak regions within a band of ±0.02 ppm either side in the 1H dimension; (2) drop the three illegitimate HSQC regions (1H > 7.0 ppm with 13C < 50 ppm; 1H < 7.0 ppm with 13C > 170 ppm; 1H < 2.4 ppm with 13C > 100 ppm); (3) exclude the shielded quadrant of the TOCSY spectrum where shifts are < 2.5 ppm — justified because peak-picking performs poorly in this crowded region and only 0.4% of predicted Natural Products Atlas molecules contain exclusively shielded 1H signals; (4) symmetrize the TOCSY data by correcting f1 chemical shifts against the corresponding f2 values; (5) exclude TOCSY signals that lack a corresponding HSQC cross-peak in the 1H dimension so only heteronuclear-anchored 1H signals feed the directed spin-system graph. The NMR solvent is selected from a drop-down menu in the GUI, which sets the solvent band location. These filters are what make subsequent 13C annotation of graph nodes and cross-sample correlation matrices reliable.

## Related tools

- **MADByTE** (Implements the filter cascade (solvent/water ±0.02 ppm band, three illegitimate HSQC regions, TOCSY < 2.5 ppm shielded quadrant, f1-vs-f2 symmetrization, HSQC cross-peak matching) as part of spin_system_construction) — https://github.com/liningtonlab/MADByTE
- **NMRglue** (Supports direct visualization of 1D spectra in the MADByTE panels so users can check peak picking and filter behavior on the underlying spectra)

## Examples

```
conda activate madbyte; python madbyte_gui.py
```

## Evaluation signals

- No retained HSQC peak falls in a solvent/water band of ±0.02 ppm in the 1H dimension or in any of the three illegitimate regions (1H > 7.0/13C < 50; 1H < 7.0/13C > 170; 1H < 2.4/13C > 100)
- No retained TOCSY signal has both F1 and F2 chemical shifts < 2.5 ppm, and every retained TOCSY 1H shift has a corresponding HSQC cross-peak in the 1H dimension
- TOCSY f1 chemical shifts match the corresponding f2 values after symmetrization (the spectrum is symmetric)
- The filtered tables feed graph construction that yields sub-graphs per unique spin system, each node annotated with exactly one closest-match 13C shift from the filtered HSQC table (closest HSQC resonance used where overlap gives multiple candidates)
- Downstream spiked-sample check: replicate the 9-sample spiked prefraction set and confirm spiked compounds (novobiocin, mupirocin, erythromycin) still form sub-clusters connected to their references by at least two shared spin system features

## Limitations

- The < 2.5 ppm shielded-quadrant exclusion means compounds with exclusively shielded 1H signals (0.4% of predicted Natural Products Atlas molecules) are precluded from analysis, and peak picking performs poorly in this crowded region
- In cases of spectral overlap producing multiple candidate 13C chemical shifts, only the closest HSQC resonance value is retained, which may mis-annotate nodes
- MADByTE will not identify every spin system in a given mixture; filtering further reduces the set of detectable features

## Evidence
