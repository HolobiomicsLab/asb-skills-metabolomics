---
name: weighted-fragment-tree-optimization
description: Use when given a low-resolution MSn peak list from a miniaturized mass
  spectrometer and a candidate formula constraint set, and you need a maximum weighted
  fragment tree to annotate top fragmentation peaks.
when_to_use_negative:
- Input spectra are high-resolution/accurate-mass data — the article notes models
  like SIRIUS4, MSNovelist, and MassGenie target high-resolution spectra and TeFT
  is not claimed to outperform them there
- Spectra were recorded in negative ion mode — TeFT was only trained on positive ion
  mode (H+) spectra
- The target molecule contains elements outside C, H, O, N, P, S, Cl, Br, I, F or
  weighs 500 Da or more
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Transformer enabled Fragment Tree (TeFT)
  role: Combined fragmentation tree models plus a deep learning Transformer module
    that generates the maximum weighted fragment tree and de novo structure from low-resolution
    MSn spectra
  repo: https://github.com/thumingo/TeFT
- name: miniaturized mass spectrometer
  role: Source instrument producing the low-resolution MSn spectra the fragment tree
    is reconstructed from
- name: SIRIUS
  role: Referenced fragmentation-tree/database-search baseline for context on fragmentation
    tree modeling
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:0297d02a015d34490871a620b0989bf6125d66115c2ec1ed60f451e92daa5ebb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - SIRIUS
  - Transformer enabled Fragment Tree (TeFT)
  - miniaturized mass spectrometer
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# weighted-fragment-tree-optimization

## Summary

Reconstructs a maximum weighted fragmentation tree directly from low-resolution MSn spectra by pre-filtering candidate molecular formulas to ten permitted elements and <500 Da, so that structures can be annotated de novo on miniaturized mass spectrometers (TeFT workflow).

## When to use

Use when given a low-resolution MSn peak list from a miniaturized mass spectrometer and a candidate formula constraint set, and you need a maximum weighted fragment tree to annotate top fragmentation peaks.

## When NOT to use

- Input spectra are high-resolution/accurate-mass data — the article notes models like SIRIUS4, MSNovelist, and MassGenie target high-resolution spectra and TeFT is not claimed to outperform them there
- Spectra were recorded in negative ion mode — TeFT was only trained on positive ion mode (H+) spectra
- The target molecule contains elements outside C, H, O, N, P, S, Cl, Br, I, F or weighs 500 Da or more

## Inputs

- Low-resolution MSn peak list from a miniaturized mass spectrometer, limited to a maximum of 100 peaks selected by ion intensity
- Spectrogram data from multiple measurements for averaging and deconvolution integration error correction

## Outputs

- Maximum weighted fragment tree built from the corrected, filtered peaks
- SMILES fragment tree capped at 1500 nodes annotating the top fragmentation peaks
- Ranked candidate substances by SMILES fragment tree similarity

## How to apply

Load the low-resolution MSn peak list capped at the top 100 peaks ranked by ion intensity, then narrow potential molecular formulas by restricting element kinds to C, H, O, N, P, S, Cl, Br, I, F and molecular weight to less than 500 Da. Before tree construction, perform error correction by averaging multiple measurements and applying deconvolution integration on the spectrogram data. Select the top five to six fragmentation peaks and their five nearby peak points as the experimental data for tree construction. Compute a reasonable loss and add weights for each fragmentation peak so the maximum weighted tree can be constructed from the low-resolution data, capping the total SMILES tree node count at 1500. Rank candidate substances by comparing SMILES fragment tree similarity between predicted molecules and the tree generated from the tandem MS spectrum.

## Related tools

- **Transformer enabled Fragment Tree (TeFT)** (Combined fragmentation tree models plus a deep learning Transformer module that generates the maximum weighted fragment tree and de novo structure from low-resolution MSn spectra) — https://github.com/thumingo/TeFT
- **miniaturized mass spectrometer** (Source instrument producing the low-resolution MSn spectra the fragment tree is reconstructed from)
- **SIRIUS** (Referenced fragmentation-tree/database-search baseline for context on fragmentation tree modeling)

## Evaluation signals

- Generated fragment tree correctly annotates the top five to six fragmentation peaks under the ten-element and <500 Da constraints
- SMILES tree output respects the 1500-node limit
- Peak input respects the 100-peak intensity cap before tree construction
- On the 16 flavonoid alcohol test set, complete structure prediction is achieved for a comparable fraction (paper reports 8/16)
- Loss/weights assigned per fragmentation peak yield a maximum weighted tree consistent with reported fragmentation rules

## Limitations

- TeFT was only trained on spectra recorded in positive ion mode (H+), restricting applicability to other ionization modes
- The model may not exhibit exceptional performance advantages on high-resolution spectra compared to other deep models
- Not separately validated with a dedicated benchmark for the tree-construction step itself; validation rests on the 16 flavonoid alcohol cases and the Anweiyiang capsule identification

## Evidence

- [discussion] constraints such as limiting the molecular weight to less than 500 Da: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] permitting only ten elements: "permitting only the presence of the ten elements C, H, O, N, P, S, Cl, Br, I, and F"
- [discussion] top peaks and nearby points selection: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [discussion] error correction via averaging and deconvolution: "we implement error correction by employing multiple measurement averaging and deconvolution integration techniques on the spectrogram data"
- [discussion] SMILES tree node cap: "we have imposed a limitation of 1500 nodes for the total count of SMILES tree nodes"
- [readme] README purpose statement: "It is aimed to generate the specific structure of molecules de novo solely from mass spectrometry spectra"
