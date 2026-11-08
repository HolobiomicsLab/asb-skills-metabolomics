---
name: low-resolution-peak-selection-and-averaging
description: Use when handling low-resolution MSn peak lists from a miniaturized mass
  spectrometer that will feed fragmentation-tree-based de novo structure prediction
  (e.g., TeFT), and you need to select, cap, and error-correct peaks.
when_to_use_negative:
- Input is high-resolution, high-accuracy MS/MS data — the article notes models like
  SIRIUS4, MSNovelist, and MassGenie are geared toward high-resolution spectra and
  may perform better there
- Spectra were recorded in negative ion mode or non-protonated species — TeFT was
  trained only on positive ion mode (H+) spectra
- The task is library matching of already-known compounds rather than de novo annotation
  of unknowns from low-resolution data
edam_topics:
- http://edamontology.org/topic_0031
tools:
- name: miniaturized mass spectrometer
  role: Source instrument producing the low-resolution MSn spectra (e.g., for on-site
    flavonoid alcohol testing) that this peak selection and averaging step preprocesses
- name: TeFT
  role: Downstream model that consumes the selected, error-corrected peaks to build
    the maximum weighted fragment tree and generate structures de novo
  repo: thumingo/TeFT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:f86bf47d692649515386cc1461722673b106da617151cb3c7d5babb41ed925c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - TeFT
  - miniaturized mass spectrometer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0031
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# low-resolution-peak-selection-and-averaging

## Summary

Preprocesses low-resolution MSn spectra from a miniaturized mass spectrometer for the TeFT fragmentation-tree workflow: peak lists are capped by ion intensity, error-corrected via multi-measurement averaging and deconvolution integration, and only the top fragmentation peaks with nearby points are retained for tree construction.

## When to use

Use when handling low-resolution MSn peak lists from a miniaturized mass spectrometer that will feed fragmentation-tree-based de novo structure prediction (e.g., TeFT), and you need to select, cap, and error-correct peaks.

## When NOT to use

- Input is high-resolution, high-accuracy MS/MS data — the article notes models like SIRIUS4, MSNovelist, and MassGenie are geared toward high-resolution spectra and may perform better there
- Spectra were recorded in negative ion mode or non-protonated species — TeFT was trained only on positive ion mode (H+) spectra
- The task is library matching of already-known compounds rather than de novo annotation of unknowns from low-resolution data

## Inputs

- Low-resolution MSn peak list from a miniaturized mass spectrometer (capped at 100 peaks by ion intensity)
- Spectrogram data from repeated measurements of the same sample (for averaging and deconvolution integration)

## Outputs

- Error-corrected peak list (top 5-6 fragmentation peaks plus five nearby peak points each) ready for maximum weighted fragment tree construction
- Filtered candidate molecular formula space (ten elements C, H, O, N, P, S, Cl, Br, I, F; molecular weight < 500 Da)

## How to apply

Start from the MSn peak list, limiting it to a maximum of 100 peaks selected by ion intensity. Apply error correction by averaging multiple measurements and performing deconvolution integration on the spectrogram data, which is necessary because low-resolution miniaturized mass spectrometers produce imprecise m/z values. Restrict downstream candidate molecular formulas to the ten elements C, H, O, N, P, S, Cl, Br, I, F with molecular weight below 500 Da. Select the top five to six fragmentation peaks together with their five nearby peak points as the experimental data from which the maximum weighted fragment tree is built; a hard limit of 1500 nodes applies to the resulting SMILES tree. The rationale is that low-resolution on-site spectra cannot support the high mass accuracy demanded by fingerprint- or library-based tools, so peak selection and correction substitute for resolution.

## Related tools

- **miniaturized mass spectrometer** (Source instrument producing the low-resolution MSn spectra (e.g., for on-site flavonoid alcohol testing) that this peak selection and averaging step preprocesses)
- **TeFT** (Downstream model that consumes the selected, error-corrected peaks to build the maximum weighted fragment tree and generate structures de novo) — thumingo/TeFT

## Evaluation signals

- The retained peak list respects the 100-peak intensity cap and contains the top 5-6 fragmentation peaks with five nearby peak points each
- Candidate formulas are constrained to the ten elements C, H, O, N, P, S, Cl, Br, I, F and molecular weight < 500 Da before tree construction
- Generated SMILES fragment tree does not exceed 1500 nodes
- The corrected peaks correctly annotate the top fragmentation peaks, matching the reported behavior in which 8 of 16 flavonoid alcohols on the miniaturized instrument achieved complete structure prediction
- Error correction demonstrably reduced peak m/z error, as verified by multi-measurement averaging followed by deconvolution integration on the spectrogram data

## Limitations

- This preprocessing step was described as part of TeFT's workflow, not separately validated with a dedicated benchmark
- Applicability is restricted to positive ion mode (H+) spectra because TeFT was trained only on such data
- The method targets low-resolution spectra with clear fragmentation rules (e.g., the flavonoid alcohol test cases); it is not claimed to outperform deep models on high-resolution spectra

## Evidence

- [discussion] we implement error correction by employing multiple measurement averaging and deconvolution integration techniques on the spectrogram data: "we implement error correction by employing multiple measurement averaging and deconvolution integration techniques on the spectrogram data"
- [discussion] the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [discussion] a peak list, limited to a maximum of 100 peaks selected based on their ion intensities: "a peak list, limited to a maximum of 100 peaks selected based on their ion intensities"
- [discussion] constraints such as limiting the molecular weight to less than 500 Da: "constraints such as limiting the molecular weight to less than 500 Da"
- [readme] TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module.: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module."
