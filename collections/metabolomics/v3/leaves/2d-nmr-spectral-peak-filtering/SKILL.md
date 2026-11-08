---
name: 2d-nmr-spectral-peak-filtering
description: Use when preparing 2D NMR peak lists (HSQC and TOCSY, e.g. from 600 MHz
  Bruker data in DMSO-d6) prior to spin-system feature construction or cross-sample
  chemical similarity networking, so that solvent, water, and physically implausible
  cross-peaks do not become spurious network nodes.
when_to_use_negative:
- When analyzing compound classes with exclusively shielded 1H signals (sterols, low
  MW isoprenoids, lipids, volatile terpenes) — the <2.5 ppm TOCSY quadrant exclusion
  precludes them from analysis
- When peaks are needed in the crowded shielded region (<2.5 ppm), where the platform's
  peak-picking performance is poor and these filters deliberately discard data
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: MADByTE
  role: Platform whose spin_system_construction step applies the solvent/water, illegitimate-region,
    shielded-quadrant, and HSQC-support filters to HSQC and TOCSY peak lists
  repo: https://github.com/liningtonlab/MADByTE
- name: NMRglue
  role: Used for 1D spectrum visualization during inspection of peaks being filtered
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jnatprod.0c01076
    title: Development of an NMR-Based Platform for the Direct Structural Annotation
      of Complex Natural Products Mixtures
  source_content_hash: sha256:499e9652482be65763b8e2f4efbc4490ed185d37725d8bae4e040f4a0c32b8e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/liningtonlab/MADByTE
  tools:
  - Gephi
  - MADByTE
  - NMRglue
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# 2D NMR spectral peak filtering

## Summary

A preprocessing procedure from the MADByTE platform that removes artefactual and uninformative peaks from HSQC and TOCSY peak lists — solvent/water regions, three illegitimate HSQC regions, the shielded TOCSY quadrant, and TOCSY signals lacking HSQC support — before spin system construction.

## When to use

Use when preparing 2D NMR peak lists (HSQC and TOCSY, e.g. from 600 MHz Bruker data in DMSO-d6) prior to spin-system feature construction or cross-sample chemical similarity networking, so that solvent, water, and physically implausible cross-peaks do not become spurious network nodes.

## When NOT to use

- When analyzing compound classes with exclusively shielded 1H signals (sterols, low MW isoprenoids, lipids, volatile terpenes) — the <2.5 ppm TOCSY quadrant exclusion precludes them from analysis
- When peaks are needed in the crowded shielded region (<2.5 ppm), where the platform's peak-picking performance is poor and these filters deliberately discard data

## Inputs

- Peak-picked HSQC peak list (1H/13C chemical shifts) from 2D NMR spectra acquired e.g. on Avance III TCI/QCI 600 MHz spectrometers
- Peak-picked TOCSY peak table with F1 and F2 1H chemical shifts
- Selected NMR solvent (drop-down choice, e.g. DMSO-d6, determining solvent/water exclusion bands)

## Outputs

- Cleaned HSQC peak list with solvent/water and illegitimate-region peaks removed
- Symmetrized, HSQC-supported TOCSY peak list (shielded quadrant and orphan TOCSY peaks excluded)
- Filtered peak tables feeding MADByTE spin system feature construction

## How to apply

Within the MADByTE workflow, after peak picking, filter the HSQC peak list to remove signals in solvent and water regions within a band of ±0.02 ppm in the 1H dimension, then delete peaks falling in the three illegitimate HSQC regions: 1H > 7.0 ppm with 13C < 50 ppm, 1H < 7.0 ppm with 13C > 170 ppm, and 1H < 2.4 ppm with 13C > 100 ppm. Symmetrize TOCSY data by correcting f1 chemical shifts against corresponding f2 values, exclude the shielded quadrant of the TOCSY spectrum (<2.5 ppm, i.e. cross-peaks where F1 and F2 are both below 2.5 ppm) because peak-picking algorithms perform poorly in that crowded region, and finally remove TOCSY signals that lack a corresponding HSQC cross-peak in the 1H dimension. The correct solvent (selected from the GUI drop-down menu) determines the solvent/water band locations. These filters are applied during spin_system_construction on each spectrum before the correlation matrix and network generation steps.

## Related tools

- **MADByTE** (Platform whose spin_system_construction step applies the solvent/water, illegitimate-region, shielded-quadrant, and HSQC-support filters to HSQC and TOCSY peak lists) — https://github.com/liningtonlab/MADByTE
- **NMRglue** (Used for 1D spectrum visualization during inspection of peaks being filtered)

## Examples

```
conda activate madbyte; python madbyte_gui.py  # select NMR solvent from drop-down, then run spin_system_construction on each HSQC/TOCSY spectrum
```

## Evaluation signals

- No HSQC peaks remain within ±0.02 ppm of the solvent or water signals in the 1H dimension
- No HSQC peaks remain in the three illegitimate regions (1H > 7.0/13C < 50; 1H < 7.0/13C > 170; 1H < 2.4/13C > 100)
- No TOCSY cross-peaks remain where both F1 and F2 are < 2.5 ppm
- Every retained TOCSY signal has a corresponding HSQC cross-peak in the 1H dimension
- Downstream validation: the filtered features reproduce the reference 17-compound network clusters (macrolides, chloramphenicols, anthracyclines) with puromycin and mupirocin unconnected

## Limitations

- Excluding the TOCSY shielded quadrant (<2.5 ppm) means compounds whose signals fall entirely in that region (0.4% of predicted Natural Products Atlas molecules) cannot be analyzed; sterols, isoprenoids, lipids, and volatile terpenes are precluded
- Peak-picking algorithms perform poorly in the crowded shielded region, motivating but also limiting the filtering
- Where spectral overlap yields multiple candidate 13C chemical shifts, the closest HSQC resonance is used, which can misassign 13C values
- Installation caveat from README: do not install MADByTE using Anaconda Navigator — use the .bat script or 'conda env create -f environment.yml'

## Evidence

- [results] filtered to remove signals in the solvent and water peak regions in a band ±0.02 ppm either side of both the solvent and water signals: "filtered to remove signals in the solvent and water peak regions in a band ±0.02 ppm either side of both the solvent and water signals"
- [results] Three illegitimate HSQC regions: "Three regions in the HSQC spectrum that do not contain legitimate signals (1H > 7.0 with 13C < 50 ppm, 1H < 7.0 with 13C > 170 ppm and 1H < 2.4 with 13C > 100 ppm) are also removed"
- [results] TOCSY signals without corresponding HSQC cross-peaks excluded: "TOCSY signals without corresponding HSQC cross-peaks in the 1H dimension are excluded"
- [discussion] Poor peak picking in crowded shielded region: "the often-poor performance of peak picking algorithms in this crowded region"
- [readme] Do not install MADByTE using Anaconda Navigator: "DO NOT INSTALL MADBYTE USING ANACONDA NAVIGATOR - IT WILL FAIL INSTALLATION."
