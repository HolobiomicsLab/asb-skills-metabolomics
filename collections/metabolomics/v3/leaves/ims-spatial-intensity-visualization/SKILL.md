---
name: ims-spatial-intensity-visualization
description: Use when you have imzML/HDF5 MALDI-MSI data and need to visualize the
  spatial distribution of a specific m/z peak — e.g. to verify that a peak correlated
  with a tumor class is genuinely elevated in the annotated tumor region.
when_to_use_negative:
- Data has already been peak-picked and reduced to a feature table with peak-intensity
  columns — the raw-spectrum channel extraction step is unnecessary
- The goal is pixel classification (tumor vs normal delineation) rather than visualizing
  a single m/z channel — use the massNet deep-learning classification skill instead
- The m/z of interest was not first established as co-localized with the region of
  interest (Pearson correlation ≥ 0.7 per the paper's msiPL selection step)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: massNet
  role: Python utilities for loading raw imzML/HDF5 MSI data and extracting m/z channels
    without prior peak picking or preprocessing
  repo: https://github.com/wabdelmoula/massNet
- name: Python
  role: Runtime environment for the ion image reconstruction (implemented with Keras/tensorflow
    stack; README specifies Python 3.6.12, h5py, numpy, scipy)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:f1302873e62ec0662283b438071d47fb384fad495132618e045f3265feaaef8a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  repo_url: https://github.com/wabdelmoula/massNet
  tools:
  - Python
  - massNet
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ims-spatial-intensity-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs a 2D ion image from raw mass spectrometry imaging data by extracting a single m/z channel (e.g. m/z 400.9546 ± 0.01 from 85,062 FT-ICR variables) and mapping ion intensities onto tissue pixel coordinates, so the spatial distribution of a candidate tumor-correlated peak can be compared against pathologist-annotated regions.

## When to use

Use when you have imzML/HDF5 MALDI-MSI data and need to visualize the spatial distribution of a specific m/z peak — e.g. to verify that a peak correlated with a tumor class is genuinely elevated in the annotated tumor region.

## When NOT to use

- Data has already been peak-picked and reduced to a feature table with peak-intensity columns — the raw-spectrum channel extraction step is unnecessary
- The goal is pixel classification (tumor vs normal delineation) rather than visualizing a single m/z channel — use the massNet deep-learning classification skill instead
- The m/z of interest was not first established as co-localized with the region of interest (Pearson correlation ≥ 0.7 per the paper's msiPL selection step)

## Inputs

- imzML or HDF5 MSI data file (e.g. Metabolomics Workbench PR001292 MALDI 9.4 Tesla FT-ICR dataset)
- m/z value of interest and tolerance window (e.g. 400.9546 ± 0.01)
- Pathologist-annotated tumor region mask transferred from a consecutive H&E-stained section

## Outputs

- 2D ion image of pixel-wise intensities at the selected m/z
- Rendered ion image with tumor annotation boundary overlay (e.g. mz400.9546_ion_image.png)
- Fold-elevation of mean ion intensity in tumor vs normal region

## How to apply

Load the imzML/HDF5 MSI data (e.g. the Metabolomics Workbench PR001292 FT-ICR GBM PDX deposit) using massNet's data-loading utilities in Python, without prior peak picking or preprocessing, consistent with massNet's raw-data approach. Select the m/z channel of interest with a tolerance window (task used 400.9546 ± 0.01 from 85,062 m/z variables). Spatially reconstruct the ion intensity at each pixel (the paper's data were acquired at 100 µm spatial resolution) into a 2D ion image matching the tissue section dimensions. Compare the ion image to the pathologist-annotated tumor region transferred from the consecutive H&E section, and quantify fold-elevation of mean tumor-region intensity versus normal-region intensity. Render the ion image with the annotation boundary overlaid.

## Related tools

- **massNet** (Python utilities for loading raw imzML/HDF5 MSI data and extracting m/z channels without prior peak picking or preprocessing) — https://github.com/wabdelmoula/massNet
- **Python** (Runtime environment for the ion image reconstruction (implemented with Keras/tensorflow stack; README specifies Python 3.6.12, h5py, numpy, scipy))

## Examples

```
import h5py; f = h5py.File('PR001292_gbm.h5','r'); mz, inten = f['mz'][:], f['intensity'][:]; mask = (mz > 400.9546-0.01) & (mz < 400.9546+0.01); ion_image = inten[:, mask].sum(axis=1).reshape(section_dims)
```

## Evaluation signals

- The rendered ion image shows visibly elevated signal within the annotated tumor region (qualitative acceptance per Figure 5e of the paper)
- Mean ion intensity inside the tumor annotation is higher than in the normal region (positive fold-elevation)
- The extracted ion image dimensions match the original tissue section dimensions at the 100 µm acquisition pixel size
- The tolerance window (±0.01) yields a single coherent ion image from the 85,062 m/z variables without needing peak picking

## Limitations

- Ground-truth annotation based solely on annotated histology is an approximation and may not be optimal, especially in cases of intra-tumor heterogeneity
- Validation is qualitative per the paper (Figure 5e); no numeric threshold for tumor-vs-normal fold-elevation is reported in the text
- Tumor-margin ground truth annotation would require more pathological investigation, so boundary overlays should be interpreted with care

## Evidence

- [results] The highest correlated peak with the tumor class was at m/z 400.9546 ± 0.01, and its ion image showed elevation in the tumor region.: [evidence span withheld — anchor status: not_found]
- [methods] Mass spec imaging acquisition at 100 µm: [evidence span withheld — anchor status: not_found]
