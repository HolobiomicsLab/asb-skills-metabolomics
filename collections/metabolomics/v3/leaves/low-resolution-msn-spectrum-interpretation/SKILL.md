---
name: low-resolution-msn-spectrum-interpretation
description: Use when MS2 spectra from a miniaturized/low-resolution ion trap instrument
  must be converted into candidate molecular structures without a library match, e.g.
  on-site identification of drug-class substances.
when_to_use_negative:
- High-resolution, high-accuracy spectra from instruments where SIRIUS4, MSNovelist,
  or MassGenie are appropriate — TeFT targets low-resolution on-site spectra
- Spectra recorded in negative ion mode — TeFT was trained only on positive ion mode
  (H+), restricting its applicability
- Compounds violating the candidate constraints (molecular weight ≥500 Da, elements
  outside C, H, O, N, P, S, Cl, Br, I, F, or SMILES trees over 1500 nodes)
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0157
tools:
- name: TeFT
  role: 'Primary model: combines fragmentation tree models and a Transformer module
    to generate molecular structures de novo from low-resolution MSn spectra'
  repo: thumingo/TeFT
- name: miniaturized mass spectrometer
  role: Instrument producing the low-resolution on-site MS2 spectra interpreted by
    the skill
- name: Transformer
  role: Deep learning module designed for generating the complete structure of the
    target substance based on MS2 spectra
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:6cf58b31a344cbc5d8f17052e4146e645f75ef3274e95f9a96f0d7cf3f58a286
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# low-resolution-msn-spectrum-interpretation

## Summary

De novo structural annotation of small molecules from low-resolution MS2/MSn spectra acquired on a miniaturized mass spectrometer, using the TeFT method (fragmentation tree models combined with a Transformer module) to generate complete SMILES structures and score them by fragment-tree similarity.

## When to use

Use when MS2 spectra from a miniaturized/low-resolution ion trap instrument must be converted into candidate molecular structures without a library match, e.g. on-site identification of drug-class substances.

## When NOT to use

- High-resolution, high-accuracy spectra from instruments where SIRIUS4, MSNovelist, or MassGenie are appropriate — TeFT targets low-resolution on-site spectra
- Spectra recorded in negative ion mode — TeFT was trained only on positive ion mode (H+), restricting its applicability
- Compounds violating the candidate constraints (molecular weight ≥500 Da, elements outside C, H, O, N, P, S, Cl, Br, I, F, or SMILES trees over 1500 nodes)

## Inputs

- Low-resolution MS2/MSn spectra recorded on a miniaturized linear ion trap mass spectrometer
- Peak lists (top 5–6 fragmentation peaks with five nearby peak points, capped at 100 peaks by ion intensity)
- Training/reference spectra compiled from GNPS, HMDB 5.0, and MoNA (positive ion mode H+)

## Outputs

- De novo SMILES structures generated from MS2 spectra
- SMILES fragment tree similarity scores used to rank and classify candidates
- Per-substance result records (compound class, prediction outcome, similarity score), e.g. table2_results.csv

## How to apply

Preprocess the low-resolution spectra by selecting the top five to six fragmentation peaks plus their five nearby peak points, capping the peak list at 100 peaks by ion intensity, and applying error correction via multiple-measurement averaging and deconvolution integration on the spectrogram data. Constrain the TeFT candidate space to molecular weight below 500 Da, only the ten elements C, H, O, N, P, S, Cl, Br, I, and F, and at most 1500 SMILES tree nodes. Run the Transformer module combined with fragmentation tree models (trained on GNPS, HMDB 5.0, and MoNA in positive ion mode H+) to generate de novo SMILES for each spectrum. Simulate molecular fragmentation of each predicted molecule and rank candidates by comparing the similarity between the SMILES fragment trees of predictions and the molecular fragment trees generated from the tandem mass spectra. Classify each substance as fully predicted based on the similarity score and aggregate outcomes by compound class, e.g. writing per-substance results to a table2_results.csv.

## Related tools

- **TeFT** (Primary model: combines fragmentation tree models and a Transformer module to generate molecular structures de novo from low-resolution MSn spectra) — thumingo/TeFT
- **miniaturized mass spectrometer** (Instrument producing the low-resolution on-site MS2 spectra interpreted by the skill)
- **Transformer** (Deep learning module designed for generating the complete structure of the target substance based on MS2 spectra)

## Evaluation signals

- Reproduce Table 2 outcomes on 23 purchased substances: 8 of 16 flavonoid alcohols, 1 of 4 stilbenes, and 1 of 2 flavones fully predicted
- Minimum fingerprint similarity per class reaches 0.98 (flavonoid alcohols), 1 (flavones), and 0.93 (stilbenes); overall minimum similarity of 0.93
- Every generated SMILES tree respects the constraints: <500 Da, only the ten permitted elements, ≤1500 nodes
- Peak lists contain at most 100 peaks selected by ion intensity, with the top 5–6 fragmentation peaks retained as experimental data
- Predicted structures remain valid after SMILES standardization and deduplication applied to the training set

## Limitations

- TeFT was trained only on spectra in positive ion mode (H+), restricting its applicability
- The model is suitable for annotating fragmentation peaks with clear fragmentation rules; performance may be limited for spectra with unclear fragmentation
- Performance on high-quality high-resolution spectra may not be exceptional compared to other deep models (SIRIUS4, MSNovelist, MassGenie)
- Requires an on-site miniaturized mass spectrometer platform where large-scale model training (like MassGenie) is not feasible

## Evidence

- [abstract] Validation on miniaturized instrument: "the TeFT has been validated on a miniaturized mass spectrometer with low-resolution spectra for 16 flavonoid alcohols, achieving complete structure prediction for 8 substances"
- [discussion] Candidate constraint thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] Peak selection as experimental data: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [discussion] Ranking by fragment tree similarity: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked"
- [readme] Method description from repository README: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module."
