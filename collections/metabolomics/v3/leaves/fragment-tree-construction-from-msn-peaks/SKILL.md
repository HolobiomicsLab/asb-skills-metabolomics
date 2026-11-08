---
name: fragment-tree-construction-from-msn-peaks
description: Use when a low-resolution MSn peak list from a miniaturized mass spectrometer
  must be turned into a maximum weighted fragmentation tree to annotate top fragmentation
  peaks and rank candidate molecule structures de novo.
when_to_use_negative:
- 'Input spectra are high-resolution/high-accuracy: TeFT was designed for low-resolution
  miniaturized mass spectrometry, and the authors note models like SIRIUS4 (with CSI:
  FingerID), MSNovelist, and MassGenie target high-resolution spectra'
- 'Spectra were recorded in negative ion mode: TeFT was only trained on spectra recorded
  in positive ion mode (H+), restricting its applicability'
- 'Candidate molecules fall outside the search space: more than the ten permitted
  elements (C, H, O, N, P, S, Cl, Br, I, F) or molecular weight >= 500 Da'
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3516
tools:
- name: TeFT
  role: Method that combines various types of fragmentation tree models with a deep
    learning Transformer module to generate the maximum weighted fragment tree from
    low-resolution MSn spectra
  repo: thumingo/TeFT
- name: miniaturized mass spectrometer
  role: Custom small linear ion trap instrument with continuous atmospheric pressure
    interface providing the low-resolution on-site MSn spectra consumed by this skill
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:3f093742715b23ad9d8d945e10cf56f2c613b2a71ee73e8e7008109063a6d261
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - TeFT
  - Transformer enabled Fragment Tree (TeFT)
  - miniaturized mass spectrometer
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3516
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# fragment-tree-construction-from-msn-peaks

## Summary

Constructs a maximum weighted fragmentation tree de novo from low-resolution MSn spectra acquired on a miniaturized mass spectrometer, following the TeFT method (Yang et al., Communications Chemistry, 2024). The tree annotates the most intense fragmentation peaks so candidate molecular structures can be ranked by fragment-tree similarity.

## When to use

Use when a low-resolution MSn peak list from a miniaturized mass spectrometer must be turned into a maximum weighted fragmentation tree to annotate top fragmentation peaks and rank candidate molecule structures de novo.

## When NOT to use

- Input spectra are high-resolution/high-accuracy: TeFT was designed for low-resolution miniaturized mass spectrometry, and the authors note models like SIRIUS4 (with CSI: FingerID), MSNovelist, and MassGenie target high-resolution spectra
- Spectra were recorded in negative ion mode: TeFT was only trained on spectra recorded in positive ion mode (H+), restricting its applicability
- Candidate molecules fall outside the search space: more than the ten permitted elements (C, H, O, N, P, S, Cl, Br, I, F) or molecular weight >= 500 Da

## Inputs

- low-resolution MSn peak list from a miniaturized mass spectrometer (capped at 100 peaks selected by ion intensity)
- raw spectrogram data for multi-measurement averaging and deconvolution integration (error correction)
- candidate molecular formula space constrained to ten elements (C, H, O, N, P, S, Cl, Br, I, F) and <500 Da

## Outputs

- maximum weighted fragmentation tree annotating the top 5-6 fragmentation peaks with their five nearby peak points
- SMILES fragment trees per candidate molecule (total SMILES tree node count limited to 1500)
- ranked list of candidate substances by SMILES fragment tree similarity against the experimental fragment tree

## How to apply

Load the low-resolution MSn peak list capped at a maximum of 100 peaks selected by ion intensity. Apply error correction first by averaging multiple measurements and performing deconvolution integration on the spectrogram data. Restrict candidate molecular formulas by permitting only the ten elements C, H, O, N, P, S, Cl, Br, I, F and limiting molecular weight to less than 500 Da. Select the top five to six fragmentation peaks, each with five nearby peak points, as the experimental data for tree construction. Build the maximum weighted fragment tree from these corrected, filtered peaks, imposing a limitation of 1500 nodes on the total SMILES tree count. The resulting trees are then used to compare the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, so substance similarity can be ranked.

## Related tools

- **TeFT** (Method that combines various types of fragmentation tree models with a deep learning Transformer module to generate the maximum weighted fragment tree from low-resolution MSn spectra) — thumingo/TeFT
- **miniaturized mass spectrometer** (Custom small linear ion trap instrument with continuous atmospheric pressure interface providing the low-resolution on-site MSn spectra consumed by this skill)

## Examples

```
git clone https://github.com/thumingo/TeFT
```

## Evaluation signals

- The generated fragment tree correctly annotates the top five to six fragmentation peaks and their five nearby peak points under the ten-element and <500 Da constraints
- Input peak list respects the cap of a maximum of 100 peaks selected by ion intensity
- Output SMILES tree does not exceed the imposed limit of 1500 total nodes
- On the validated miniaturized-MS benchmark of 16 flavonoid alcohols, complete structure prediction is achieved for 8 substances
- Ranking by SMILES fragment tree similarity recovers the known compound (e.g., the Anweiyang capsule compound) among the top candidates

## Limitations

- TeFT was only trained on spectra recorded in positive ion mode (H+), restricting its applicability to negative ion mode data
- The model may not exhibit exceptional performance advantages in predicting high-quality resolution spectra compared to other deep models
- Applying this step is described as part of the method's workflow, not separately validated with a dedicated benchmark; the full list of dissociation and rearrangement rules is in Supplementary Table S1 without versioning or public availability details
- The custom-made small linear ion trap mass spectrometer has no release/distribution plan, limiting reproducibility of the on-site spectra

## Evidence

- [discussion] permitting only the presence of the ten elements C, H, O, N, P, S, Cl, Br, I, and F: "permitting only the presence of the ten elements C, H, O, N, P, S, Cl, Br, I, and F"
- [discussion] the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [discussion] we implement error correction by employing multiple measurement averaging and deconvolution integration techniques on the spectrogram data: "we implement error correction by employing multiple measurement averaging and deconvolution integration techniques on the spectrogram data"
- [readme] TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module.: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module."
- [discussion] we have imposed a limitation of 1500 nodes for the total count of SMILES tree nodes: "we have imposed a limitation of 1500 nodes for the total count of SMILES tree nodes"
