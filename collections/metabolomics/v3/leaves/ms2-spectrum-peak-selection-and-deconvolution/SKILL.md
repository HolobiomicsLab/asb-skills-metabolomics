---
name: ms2-spectrum-peak-selection-and-deconvolution
description: Use when MS2 spectra come from a miniaturized, low-resolution mass spectrometer
  and must be reduced to a compact, error-corrected peak list before feeding a de
  novo structure-prediction model such as TeFT.
when_to_use_negative:
- 'High-resolution, high-accuracy spectra: the article notes models like MassGenie
  and MSNovelist are geared to high-resolution/high-accuracy spectra, and TeFT''s
  advantage is targeted at low-resolution on-site spectra'
- Spectra dominated by peaks without clear fragmentation rules — TeFT is described
  as suitable for annotating fragmentation peaks with clear fragmentation rules, so
  a deconvolved peak list will not help where rules are absent
- 'Negative ion mode data: TeFT was only trained on positive ion mode (H+) spectra,
  so preprocessed peaks from negative-mode runs lack a trained downstream consumer'
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3316
tools:
- name: TeFT
  role: Downstream consumer of the selected and deconvolved peak list; the compact
    peak set is the experimental data from which TeFT generates de novo SMILES structures
  repo: thumingo/TeFT
- name: miniaturized mass spectrometer
  role: Source instrument producing the low-resolution MS2 spectra and repeated measurements
    that this preprocessing step reduces and error-corrects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:eaa69060627e5f2e0ab08578c5ad5cebeed8b54dca9ca60ba1464748b8c53988
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - GNPS
  - HMDB (5.0)
  - MoNA
  - TeFT
  - Transformer
  - miniaturized mass spectrometer
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# MS2 spectrum peak selection and deconvolution

## Summary

Preprocessing step for low-resolution MS2 spectra recorded on a miniaturized linear ion trap mass spectrometer: select the top ion-intensity fragmentation peaks and apply measurement averaging and deconvolution integration to correct errors before de novo structure prediction with TeFT.

## When to use

Use when MS2 spectra come from a miniaturized, low-resolution mass spectrometer and must be reduced to a compact, error-corrected peak list before feeding a de novo structure-prediction model such as TeFT.

## When NOT to use

- High-resolution, high-accuracy spectra: the article notes models like MassGenie and MSNovelist are geared to high-resolution/high-accuracy spectra, and TeFT's advantage is targeted at low-resolution on-site spectra
- Spectra dominated by peaks without clear fragmentation rules — TeFT is described as suitable for annotating fragmentation peaks with clear fragmentation rules, so a deconvolved peak list will not help where rules are absent
- Negative ion mode data: TeFT was only trained on positive ion mode (H+) spectra, so preprocessed peaks from negative-mode runs lack a trained downstream consumer

## Inputs

- Low-resolution MS2 tandem mass spectra from a miniaturized linear ion trap mass spectrometer
- Raw spectrogram data from multiple repeated measurements of the same substance
- Positive-mode (H+) ion spectra of small-molecule substances (e.g. flavanols, flavones, stilbenes)

## Outputs

- Error-corrected peak list (maximum 100 peaks ranked by ion intensity) per substance
- Selected top 5–6 fragmentation peaks with their five nearby peak points, as TeFT-ready experimental input
- Cleaned spectrogram data suitable for fragment-tree similarity ranking of predicted SMILES candidates

## How to apply

From each low-resolution tandem MS2 spectrum, select the top five to six fragmentation peaks and their five nearby peak points as the experimental data. Cap the resulting peak list at a maximum of 100 peaks, ranked by ion intensity. Because the miniaturized instrument yields noisy, low-resolution spectrograms, correct errors by averaging multiple measurements and applying deconvolution integration on the spectrogram data. The rationale is that downstream de novo structure generation (TeFT, trained on GNPS, HMDB 5.0 and MoNA in positive H+ mode) and fragment-tree similarity ranking work from a compact peak set representing the dominant fragmentation events rather than the full spectrum.

## Related tools

- **TeFT** (Downstream consumer of the selected and deconvolved peak list; the compact peak set is the experimental data from which TeFT generates de novo SMILES structures) — thumingo/TeFT
- **miniaturized mass spectrometer** (Source instrument producing the low-resolution MS2 spectra and repeated measurements that this preprocessing step reduces and error-corrects)

## Evaluation signals

- Peak list length per spectrum must not exceed 100 peaks after intensity-based selection
- Selected experimental data must contain the top 5–6 fragmentation peaks plus their five nearby peak points per spectrum
- Repeated measurements of the same substance must be combined via averaging and deconvolution integration before scoring, not used as independent spectra
- Downstream check: after TeFT prediction and fragment-tree similarity ranking on the preprocessed peaks, per-class minimum similarity should reach ~0.93–1.00 (Table 2: minima 0.98, 1, 0.93 for flavanols, flavones, stilbenes)
- Preprocessed peaks must be in positive ion mode (H+) to match the TeFT training data

## Limitations

- TeFT (and thus this preprocessing target) was only trained on spectra recorded in positive ion mode (H+), restricting applicability to negative-mode acquisitions
- Low-resolution spectrograms from the miniaturized instrument require multiple-measurement averaging and deconvolution integration; single-shot spectra carry uncorrected errors
- The method works best for fragmentation peaks with clear fragmentation rules; unclear-rule peaks are not well annotated even after peak selection

## Evidence

- [discussion] Top peaks selection: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [discussion] Peak list cap: "a peak list, limited to a maximum of 100 peaks selected based on their ion intensities"
- [discussion] Error correction via averaging and deconvolution: "we implement error correction by employing multiple measurement averaging and deconvolution integration techniques on the spectrogram data"
- [readme] Downstream purpose (TeFT model): "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module."
