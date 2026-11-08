---
name: segmentation-and-colocalisation-feature-comparison
description: Use when you must decide which LA-ICP-MS imaging software to adopt for
  vendor-native import, or when you need to segment element images and quantify Mn/Zn-type
  spatial colocalisation.
when_to_use_negative:
- Your data is already a calibrated feature/colocalisation table and only needs visualisation,
  not import or segmentation.
- You need native import of a vendor format other than Agilent, Perkin Elmer, Thermo,
  or Nu Instruments and cannot convert to delimited text first — the article notes
  most open-source alternatives require preformatted data.
- The element of interest is absent from most of the sample, breaking the marker-element
  peak integration requirement for spot-wise data.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Pew2
  role: GUI for drag-and-drop vendor import, drift/spike correction, segmentation
    and RGB colocalisation of LA-ICP-MS images
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library behind Pew2 for import/export and batch processing of LA-ICP-MS
    data
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: underlying numerical library used by the Pew2 implementation
- name: PySide2
  role: cross-platform Qt5 GUI toolkit for the Pew2 interface
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:628d391bcc82fda5442804cf1d1ce113412509476fc0d23caa5a2fbb02f2cdad
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - PySide2
  - pandas
  - pewlib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# segmentation-and-colocalisation-feature-comparison

## Summary

Builds a feature-by-software capability matrix for LA-ICP-MS imaging programs and then applies Pew2's built-in segmentation (k-means, mean/Otsu thresholding) and RGB colocalisation (Pearson/Li-ICQ) to element images such as 55Mn, 56Fe and 66Zn.

## When to use

Use when you must decide which LA-ICP-MS imaging software to adopt for vendor-native import, or when you need to segment element images and quantify Mn/Zn-type spatial colocalisation.

## When NOT to use

- Your data is already a calibrated feature/colocalisation table and only needs visualisation, not import or segmentation.
- You need native import of a vendor format other than Agilent, Perkin Elmer, Thermo, or Nu Instruments and cannot convert to delimited text first — the article notes most open-source alternatives require preformatted data.
- The element of interest is absent from most of the sample, breaking the marker-element peak integration requirement for spot-wise data.

## Inputs

- raw Agilent MassHunter batch folders (.b directories, G7200B or later) with one sample per line
- Perkin Elmer Elan '.xl' directories
- Thermo iCap CSV / LDR exports or Qtegra exports
- delimited 2D text image files and one-file-per-csv line data
- Nu Instruments Vitesse images and imzML data (import wizard)
- Iolite laser logs (ActiveView2 or Chromium3 style) for syncing with ICP data

## Outputs

- feature-by-software comparison matrix (TSV) with capability rows and software columns
- segmentation masks for ROIs/background (k-means, mean, or Otsu)
- RGB overlay image of up to three element channels
- colocalisation table of Pearson / Li-ICQ values between element pairs

## How to apply

First enumerate capabilities from the article text and Table 1 software list, then build a rectangular pandas matrix with capabilities as rows and packages (Pew2, HDIP, Iolite, LA-iMageS, etc.) as columns, writing a TSV with a header and capability column; leave cells blank where the article does not state a feature rather than guessing. For the imaging part, drag-and-drop import raw Agilent MassHunter G7200B+ batches, Perkin Elmer Elan 'XL' or Thermo Qtegra exports directly (or flexible text image / file-per-line exports), enter laser spot size and scan speed manually after import, and run drift renormalisation (polynomial fit in a user-defined background slice) and local mean/median spike filtering with a user-defined threshold before analysis. Segment regions of interest either manually or automatically with three-level k-means, mean thresholding, or Otsu's method, using the resulting masks to remove background or select image features. Finally overlay up to three elements as RGB channels (e.g. 55Mn red, 56Fe green, 66Zn blue) and read colocalisation statistics from the software's Pearson/Li-ICQ output, cross-checking image magenta overlap with the numeric table.

## Related tools

- **Pew2** (GUI for drag-and-drop vendor import, drift/spike correction, segmentation and RGB colocalisation of LA-ICP-MS images) — https://github.com/djdt/pewpew
- **pewlib** (Python library behind Pew2 for import/export and batch processing of LA-ICP-MS data) — https://github.com/djdt/pewlib
- **NumPy** (underlying numerical library used by the Pew2 implementation)
- **PySide2** (cross-platform Qt5 GUI toolkit for the Pew2 interface)

## Examples

```
git clone https://github.com/djdt/pewpew && cd pewpew && pip install -r requirements.txt && pip install -e . && pewpew
```

## Evaluation signals

- TSV parses to a rectangular table; every extracted Pew2 capability plus all Table 1 software columns appear as columns; blank cells only where the article is silent.
- Calibration r² values match Table 2 ranges (55Mn 0.9931, 56Fe 0.9959, 63Cu 0.9940, 66Zn 0.9915) if calibration is reproduced.
- Segmentation masks for 56Fe via three-level k-means, mean thresholding and Otsu's method each isolate the same tissue ROI from background as in Figure 5.
- RGB overlay of 55Mn/56Fe/66Zn shows Mn–Zn overlap as magenta regions consistent with Figure 6.
- Colocalisation values in the Pearson/Li-ICQ table agree with the software's numeric output (article Table 3 correlative Mn–Zn relationship).

## Limitations

- Spot-wise import requires at least one element present across the entire sample with similar peak positions for each element.
- Feature cells for competing software packages must be left blank when the article does not state them — no feature inference beyond reported values.
- Spike removal and drift renormalisation depend on user-defined thresholds/background slices; incorrect settings leave instrument-noise artifacts in ROIs.

## Evidence

- [readme] pewlib supports Agilent, Thermo, PerkinElmer and delimited text images: [evidence span withheld — anchor status: external_doc]
