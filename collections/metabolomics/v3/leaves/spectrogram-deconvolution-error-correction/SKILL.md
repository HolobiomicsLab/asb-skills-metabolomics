---
name: spectrogram-deconvolution-error-correction
description: Use when low-resolution MSn spectra from a miniaturized mass spectrometer
  feed de novo fragment-tree or formula prediction and raw mass errors would corrupt
  constraints such as the <500 Da cutoff or ten-element formula search.
when_to_use_negative:
- High-resolution, high-accuracy spectra where the instrument's mass accuracy already
  satisfies formula-annotation constraints without deconvolution correction
- Spectra recorded in negative ion mode — TeFT was trained only on positive ion mode
  (H+) spectra, so corrected peaks would leave the model's applicability domain
- Peak lists already deconvoluted and error-corrected upstream; re-applying averaging/deconvolution
  could distort intensities
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TeFT
  role: Pipeline whose front-end error-correction step this skill implements; corrected
    spectrogram feeds TeFT's fragment-tree and Transformer structure-generation modules
  repo: https://github.com/thumingo/TeFT
- name: miniaturized mass spectrometer
  role: Source instrument producing the low-resolution MSn spectrogram data that require
    averaging and deconvolution integration correction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:00a6a02c8f580a1601d02fc6e472faaccaa701403632aa4f82e0cdb834d1d7f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - TeFT
  - miniaturized mass spectrometer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectrogram-deconvolution-error-correction

## Summary

An error-correction preprocessing step from the TeFT workflow that compensates for mass inaccuracies of low-resolution miniaturized-MS spectra by averaging multiple measurements and applying deconvolution integration on the spectrogram data before fragment-tree construction.

## When to use

Use when low-resolution MSn spectra from a miniaturized mass spectrometer feed de novo fragment-tree or formula prediction and raw mass errors would corrupt constraints such as the <500 Da cutoff or ten-element formula search.

## When NOT to use

- High-resolution, high-accuracy spectra where the instrument's mass accuracy already satisfies formula-annotation constraints without deconvolution correction
- Spectra recorded in negative ion mode — TeFT was trained only on positive ion mode (H+) spectra, so corrected peaks would leave the model's applicability domain
- Peak lists already deconvoluted and error-corrected upstream; re-applying averaging/deconvolution could distort intensities

## Inputs

- Low-resolution MSn spectrogram data from a miniaturized (small linear ion trap) mass spectrometer
- Multiple replicate measurements of the same sample (for measurement averaging)

## Outputs

- Error-corrected spectrogram/peak list (≤100 peaks ranked by ion intensity) ready for fragment-tree construction

## How to apply

Load the low-resolution MSn spectrogram and, before any peak selection or formula filtering, perform error correction by combining multiple measurement averaging with deconvolution integration on the spectrogram data. This stabilizes peak m/z values acquired by the miniaturized mass spectrometer so that downstream steps — selecting the top 5–6 fragmentation peaks with their five nearby peak points, capping the peak list at 100 peaks by ion intensity, and restricting formulas to C, H, O, N, P, S, Cl, Br, I, F with molecular weight below 500 Da — operate on corrected peaks. The corrected spectrogram is the input for building the maximum weighted fragmentation tree (up to 1500 SMILES tree nodes). Rationale: uncorrected low-resolution peak shifts would violate the element/mass constraints and mis-weight tree edges, so the article places this correction at the front of the TeFT workflow rather than validating it separately.

## Related tools

- **TeFT** (Pipeline whose front-end error-correction step this skill implements; corrected spectrogram feeds TeFT's fragment-tree and Transformer structure-generation modules) — https://github.com/thumingo/TeFT
- **miniaturized mass spectrometer** (Source instrument producing the low-resolution MSn spectrogram data that require averaging and deconvolution integration correction)

## Evaluation signals

- Corrected peaks let the top 5–6 fragmentation peaks (with their five nearby peak points each) be consistently annotated across replicate measurements
- Peak m/z values after correction are compatible with the ten-element (C,H,O,N,P,S,Cl,Br,I,F) and <500 Da formula constraints without spurious rejections
- On the 16-flavonoid-alcohol miniaturized-MS validation set, corrected spectra should support complete structure prediction for the expected fraction of substances (8 of 16 in the reference study)
- Total SMILES tree nodes for the constructed fragment tree stay within the imposed 1500-node limit

## Limitations

- The article describes this correction as part of the TeFT workflow, not separately validated with a dedicated benchmark
- Correction is designed for low-resolution spectra of small molecules (<500 Da, ten permitted elements); it does not extend applicability beyond TeFT's positive-ion-mode (H+) training data
- Effective only when multiple replicate measurements of the same sample are available, since averaging is part of the correction

## Evidence

- [discussion] we implement error correction by employing multiple measurement averaging and deconvolution integration techniques on the spectrogram data: "we implement error correction by employing multiple measurement averaging and deconvolution integration techniques on the spectrogram data"
- [results] Error correction is applied beforehand: [evidence span withheld — anchor status: not_found]
- [readme] It is aimed to generate the specific structure: "It is aimed to generate the specific structure of molecules de novo solely from mass spectrometry spectra"
