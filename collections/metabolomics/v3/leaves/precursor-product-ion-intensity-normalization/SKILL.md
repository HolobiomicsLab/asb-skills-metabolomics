---
name: precursor-product-ion-intensity-normalization
description: Use when MS-DIAL result tables contain raw precursor and fragment ion
  intensities that must be converted to comparable 0–1 scales before rendering MS1×MS2
  grayscale fingerprint images for CNN input.
when_to_use_negative:
- Input is already an aligned feature table destined for PCA, OPLS-DA, or Random Forest
  — those baselines consume the feature matrix directly, not the 0–1 normalized image-encoding
  values
- Data comes from a non-MS-DIAL preprocessing pipeline where product-ion intensities
  are not grouped by precursor ion, since the 0–1 product scaling is defined relative
  to product ions of the same precursor
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS-DIAL
  role: Preprocesses raw LC-MS/MS data and extracts the precursor (MS1) and fragment
    (MS2) m/z pairs with intensities that are then normalized
- name: Python 3.10
  role: Implements the intensity scaling (pandas for the MS-DIAL result table) prior
    to image rendering
- name: matplotlib
  role: Consumes the normalized intensities to render the grayscale scatter image,
    precursor intensity as point size and product-ion intensity as grayscale color
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:c4770bb7d82401bbde1307542244d77d7a4c28d916519e8ad1eabba55f8681a4
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MS-DIAL
  - Python
  - Python 3.10
  - TensorFlow
  - matplotlib
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# precursor-product-ion-intensity-normalization

## Summary

Normalizes MS-DIAL-extracted precursor (MS1) ion intensities to a 0–1 global range and product-ion (MS2) intensities to a 0–1 range relative to product ions of the same precursor, so they can be encoded as size and grayscale intensity in the MS2toImg molecular fingerprint scatter image.

## When to use

Use when MS-DIAL result tables contain raw precursor and fragment ion intensities that must be converted to comparable 0–1 scales before rendering MS1×MS2 grayscale fingerprint images for CNN input.

## When NOT to use

- Input is already an aligned feature table destined for PCA, OPLS-DA, or Random Forest — those baselines consume the feature matrix directly, not the 0–1 normalized image-encoding values
- Data comes from a non-MS-DIAL preprocessing pipeline where product-ion intensities are not grouped by precursor ion, since the 0–1 product scaling is defined relative to product ions of the same precursor

## Inputs

- MS-DIAL result txt file with precursor (MS1) and fragment (MS2) m/z pairs and their corresponding intensities per sample
- Peak intensity table filtered to values > 10⁻⁴ (TIC-normalized, adduct set [M-H]⁻, [M-H₂O-H]⁻, [M+Na-2H]⁻, [M-K-2H]⁻, [M-FA-H]⁻)

## Outputs

- Per-sample arrays of precursor ion intensities scaled to 0–1
- Per-precursor product-ion intensities scaled to 0–1 relative to product ions of the same precursor
- Normalized intensity table ready for grayscale scatter-image rendering (matplotlib, 600 dpi)

## How to apply

Load the MS-DIAL result txt file and extract four fields per sample: precursor MS1 m/z, precursor ion intensity, product MS2 m/z values, and product ion intensities (Python/pandas). Normalize precursor ion intensities to the 0–1 range globally across the sample. For product ions, scale each intensity between 0 and 1 with respect to the intensities of product ions belonging to the same precursor ion, so relative fragmentation abundance is preserved within each precursor's fragment group. Retain only peak intensity values greater than 10⁻⁴ before scaling, since MS-DIAL output was pre-filtered on this threshold. The normalized precursor intensity later drives point size and normalized product-ion intensity drives grayscale color intensity in the matplotlib scatter plot (MS1 m/z on x, MS2 m/z on y, 600 dpi), with peaks above the MS2 > MS1 diagonal masked to zero.

## Related tools

- **MS-DIAL** (Preprocesses raw LC-MS/MS data and extracts the precursor (MS1) and fragment (MS2) m/z pairs with intensities that are then normalized)
- **Python 3.10** (Implements the intensity scaling (pandas for the MS-DIAL result table) prior to image rendering)
- **matplotlib** (Consumes the normalized intensities to render the grayscale scatter image, precursor intensity as point size and product-ion intensity as grayscale color)

## Examples

```
import pandas as pd; df = pd.read_csv('msdial_result.txt', sep='\t'); df['prec_norm'] = df['prec_intensity'] / df['prec_intensity'].max(); df['prod_norm'] = df.groupby('prec_mz')['prod_intensity'].transform(lambda x: x / x.max())
```

## Evaluation signals

- All normalized precursor intensities fall within [0, 1]
- Within each precursor group, the maximum product-ion intensity equals 1 and the minimum approaches 0
- Every peak intensity fed into scaling exceeds the 10⁻⁴ retention threshold
- Downstream rendered images show precursor intensity as point size and product intensity as grayscale brightness, with the MS2 > MS1 region entirely black
- One grayscale PNG fingerprint image exists per sample at 600 dpi

## Limitations

- Product-ion normalization is relative within a precursor group, so absolute fragment abundance across precursors is not comparable in the encoded grayscale values
- Pipeline assumes MS-DIAL output with TIC normalization and the specified negative-mode adduct set; other acquisition modes or adducts were not validated
- Normalized values are intermediate to image rendering; the repo README does not provide a standalone normalization entry point

## Evidence
