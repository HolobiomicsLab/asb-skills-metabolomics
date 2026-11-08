---
name: low-resolution-ms2-spectrum-peak-annotation
description: Use when a low-resolution MS2 spectrum (e.g., from a miniaturized mass
  spectrometer) has top fragmentation peaks that need structural annotation, and no
  library match or negative standard is available.
when_to_use_negative:
- High-resolution, high-mass-accuracy spectra where models like SIRIUS4 (CSI:FingerID),
  MSNovelist, or MassGenie are better suited
- Spectra recorded in negative ion mode — TeFT was trained only on positive ion mode
  (H+) spectra
- Molecules heavier than 500 Da or containing elements outside the ten permitted element
  types
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: TeFT
  role: Runs fragmentation tree simulation and the Transformer module to generate
    candidate structures de novo from the low-resolution MS2 spectrum and rank them
    by SMILES fragment tree similarity
  repo: https://github.com/thumingo/TeFT
- name: miniaturized mass spectrometer
  role: Acquires the low-resolution MS2 spectrum of the sample extract (e.g., ultrasound-assisted
    flavonoid extraction analyzed on a custom small linear ion trap instrument)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:21f03ed5c28762344565eab0da942f9d9a16bc2015a7402835c213b5cc434599
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - TeFT
  - miniaturized mass spectrometer
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# low-resolution-ms2-spectrum-peak-annotation

## Summary

Annotates fragmentation peaks in low-resolution MS2 spectra from miniaturized mass spectrometers by simulating fragmentation trees with TeFT and ranking candidate structures by SMILES fragment tree similarity. Designed for on-site annotation where library search and high-resolution de novo models fail.

## When to use

Use when a low-resolution MS2 spectrum (e.g., from a miniaturized mass spectrometer) has top fragmentation peaks that need structural annotation, and no library match or negative standard is available.

## When NOT to use

- High-resolution, high-mass-accuracy spectra where models like SIRIUS4 (CSI:FingerID), MSNovelist, or MassGenie are better suited
- Spectra recorded in negative ion mode — TeFT was trained only on positive ion mode (H+) spectra
- Molecules heavier than 500 Da or containing elements outside the ten permitted element types

## Inputs

- Low-resolution MS2 spectrum from a miniaturized mass spectrometer (e.g., m/z 269 marker peak from Anweiyang capsule extract)
- Peak list of up to 100 peaks by ion intensity, with top 5-6 fragmentation peaks and their five nearby peak points
- Candidate molecular structures as SMILES strings (MW < 500 Da, elements limited to C, H, O, N, P, S, Cl, Br, I, F)

## Outputs

- Ranked list of candidate molecular structures by SMILES fragment tree similarity
- Annotation of selected fragmentation peaks (fragment tree assignments) for the top-ranked candidate

## How to apply

Preprocess the spectrogram by multiple measurement averaging and deconvolution integration for error correction, then build a peak list limited to a maximum of 100 peaks selected by ion intensity; select the top five to six fragmentation peaks and their five nearby peak points as experimental data. Run TeFT's fragmentation tree simulation, which combines fragmentation tree models with a Transformer module trained on GNPS, HMDB 5.0, and MoNA, to generate candidate structures from the spectrum. Constrain candidate molecular formulas by permitting only the ten elements C, H, O, N, P, S, Cl, Br, I, F and limiting molecular weight to less than 500 Da. Rank candidates by comparing SMILES fragment tree similarity between each predicted molecule and the molecular fragment trees generated from the tandem mass spectrometry data, imposing a limitation of 1500 nodes for the SMILES tree. Validate by checking whether the top-ranked candidate (e.g., formononetin for the Anweiyang capsule m/z 269 marker) fully annotates all selected fragmentation peaks with clear fragmentation rules.

## Related tools

- **TeFT** (Runs fragmentation tree simulation and the Transformer module to generate candidate structures de novo from the low-resolution MS2 spectrum and rank them by SMILES fragment tree similarity) — https://github.com/thumingo/TeFT
- **miniaturized mass spectrometer** (Acquires the low-resolution MS2 spectrum of the sample extract (e.g., ultrasound-assisted flavonoid extraction analyzed on a custom small linear ion trap instrument))

## Examples

```
git clone https://github.com/thumingo/TeFT
```

## Evaluation signals

- The highest-scoring candidate must fully annotate all selected fragmentation peaks (e.g., formononetin supplied complete annotations for three of the fragmentation peaks in the m/z 269 spectrum)
- All candidate formulas comply with the element filter (only C, H, O, N, P, S, Cl, Br, I, F) and the MW < 500 Da constraint
- SMILES fragment trees of ranked candidates stay within the 1500-node limit
- Peak list contains at most 100 peaks selected by ion intensity, with the top 5-6 fragmentation peaks plus five nearby peak points annotated
- On the flavonoid benchmark (16 flavonoid alcohols), the pipeline is expected to achieve complete structure prediction for about 8 of 16 substances

## Limitations

- TeFT was trained only on positive ion mode (H+) spectra, restricting applicability to negative ion mode data
- Best suited to fragmentation peaks with clear fragmentation rules; performance on compounds with unclear fragmentation pathways is not guaranteed
- The model may not exhibit exceptional performance advantages in predicting high-quality high-resolution spectra compared to other deep models like MassGenie or SIRIUS4
- Requires error correction (averaging and deconvolution integration) on the raw spectrogram because miniaturized instruments yield low-resolution data

## Evidence

- [abstract] Fragmentation peak annotation scope: "the TeFT method is suitable for annotating fragmentation peaks with clear fragmentation rules, particularly when applied to on-site mass spectrometry with lower mass"
- [discussion] Experimental peak selection: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [discussion] Peak list intensity limit: "a peak list, limited to a maximum of 100 peaks selected based on their ion intensities"
- [discussion] Molecular weight and element constraints: "constraints such as limiting the molecular weight to less than 500 Da"
- [readme] TeFT model definition: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module."
