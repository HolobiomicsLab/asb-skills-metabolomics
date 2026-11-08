---
name: molecular-formula-constraint-filtering
description: Use when preparing low-resolution MSn peak lists from a miniaturized
  mass spectrometer for fragment tree construction and you must prune candidate molecular
  formulas by element kinds and weight before tree building.
when_to_use_negative:
- Input spectra are high-resolution/high-accuracy—models like SIRIUS4, MSNovelist,
  or MassGenie are geared toward such data, and TeFT's constraints are designed for
  low-resolution miniaturized instrument spectra.
- Target compounds contain elements outside the ten permitted kinds (e.g. Si, metals,
  isotopes not among C, H, O, N, P, S, Cl, Br, I, F).
- Spectra were recorded in negative ion mode—TeFT was only trained on positive ion
  mode (H+) spectra.
edam_topics:
- http://edamontology.org/topic_0031
tools:
- name: miniaturized mass spectrometer
  role: Produces the low-resolution MSn peak lists whose narrow mass accuracy motivates
    the formula constraint filtering
- name: Transformer enabled Fragment Tree (TeFT)
  role: Consumes the filtered, corrected peak subset to build the maximum weighted
    fragment tree
  repo: https://github.com/thumingo/TeFT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:6303bf96cef1b99180e0451d018097cee085b5f2a59160377683b56d9e848c26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - Transformer enabled Fragment Tree (TeFT)
  - miniaturized mass spectrometer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0031
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# molecular-formula-constraint-filtering

## Summary

Applies TeFT's pre-tree candidate constraints—limiting elements to C, H, O, N, P, S, Cl, Br, I, F and molecular weight to <500 Da—to narrow molecular formula search space before building a maximum weighted fragmentation tree from low-resolution MSn spectra.

## When to use

Use when preparing low-resolution MSn peak lists from a miniaturized mass spectrometer for fragment tree construction and you must prune candidate molecular formulas by element kinds and weight before tree building.

## When NOT to use

- Input spectra are high-resolution/high-accuracy—models like SIRIUS4, MSNovelist, or MassGenie are geared toward such data, and TeFT's constraints are designed for low-resolution miniaturized instrument spectra.
- Target compounds contain elements outside the ten permitted kinds (e.g. Si, metals, isotopes not among C, H, O, N, P, S, Cl, Br, I, F).
- Spectra were recorded in negative ion mode—TeFT was only trained on positive ion mode (H+) spectra.

## Inputs

- Low-resolution MSn peak list from a miniaturized mass spectrometer, capped at 100 peaks by ion intensity
- Raw spectrogram data requiring averaging and deconvolution-integration error correction

## Outputs

- Filtered candidate molecular formula set (ten-element CHONPSClBrIF restriction, molecular weight <500 Da)
- Corrected, filtered peak subset (top 5-6 fragmentation peaks with five nearby peak points each) feeding the maximum weighted fragment tree

## How to apply

Start from the low-resolution MSn peak list (capped at 100 peaks by ion intensity). Restrict candidate molecular formulas to the ten elements C, H, O, N, P, S, Cl, Br, I, and F and impose the <500 Da molecular weight cutoff, also restricting the kinds and quantities of each element. Before tree construction, apply error correction via multiple measurement averaging and deconvolution integration on the spectrogram data. Only after filtering and correction, select the top five to six fragmentation peaks (each with five nearby peak points) as experimental data and build the maximum weighted fragment tree. A global budget of 1500 SMILES tree nodes bounds downstream structure generation, so keep the filtered candidate space small enough to stay within it.

## Related tools

- **miniaturized mass spectrometer** (Produces the low-resolution MSn peak lists whose narrow mass accuracy motivates the formula constraint filtering)
- **Transformer enabled Fragment Tree (TeFT)** (Consumes the filtered, corrected peak subset to build the maximum weighted fragment tree) — https://github.com/thumingo/TeFT

## Evaluation signals

- Every retained candidate formula uses only elements from {C, H, O, N, P, S, Cl, Br, I, F}
- No candidate formula exceeds the 500 Da molecular weight limit
- Input peak list does not exceed 100 peaks ranked by ion intensity
- Error correction (multi-measurement averaging plus deconvolution integration) is completed on the spectrogram before tree construction
- Generated SMILES fragment tree stays within the 1500-node total budget while annotating the top fragmentation peaks

## Limitations

- TeFT was trained only on spectra recorded in positive ion mode (H+), restricting applicability.
- The model may not show exceptional performance advantages on high-quality resolution spectra compared to other deep models.
- No quantitative performance comparison with SIRIUS4/MSNovelist/MassGenie is provided for the filtering/constraint stage.

## Evidence

- [discussion] permitting only the presence of the ten elements C, H, O, N, P, S, Cl, Br, I, and F: "permitting only the presence of the ten elements C, H, O, N, P, S, Cl, Br, I, and F"
- [discussion] constraints such as limiting the molecular weight to less than 500 Da: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] a peak list, limited to a maximum of 100 peaks selected based on their ion intensities: "a peak list, limited to a maximum of 100 peaks selected based on their ion intensities"
- [intro] Due to the chemical diversity of these compounds’ structures and the limited mass resolution of miniaturized mass spectrometers, on-site determination: "Due to the chemical diversity of these compounds’ structures and the limited mass resolution of miniaturized mass spectrometers"
