---
name: fingerprint-linear-probing-on-mass-spectra
description: Use when you have pre-trained spectral embeddings (e.g. from DreaMS on
  GeMS HDF5 spectra) and want to test whether the representation space encodes molecular
  substructure without fine-tuning the encoder.
when_to_use_negative:
- You need production-quality structure annotation from spectra — the authors state
  fine-tuned DreaMS fingerprint prediction 'should be understood as a proof of concept',
  and probing is diagnostic, not an annotation tool
- You want to predict fingerprints from raw spectra without a pre-trained encoder
  — use a supervised fingerprint predictor such as MIST or MIST-CF instead
- The input is already a labeled feature table for supervised classification — a linear
  probe assumes an unlabeled self-supervised representation being evaluated, not a
  supervised modeling pipeline
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0157
tools:
- name: DreaMS
  role: Source of the frozen 1,024-dimensional self-supervised spectral embeddings
    probed for MACCS bits; code and GeMS data from the pluskal-lab/DreaMS repository
  repo: https://github.com/pluskal-lab/DreaMS
- name: MIST
  role: Supervised fingerprint-prediction baseline used to contextualize how much
    structure a linear probe recovers from the self-supervised representation
  repo: https://github.com/samgoldman97/mist
- name: MS2DeepScore
  role: Supervised embedding baseline; DreaMS cosine similarity in embedding space
    outperforms it, supporting the quality of the probed representations
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-025-02663-3
    title: Self-supervised learning of molecular representations from millions of
      tandem mass spectra using DreaMS
  source_content_hash: sha256:e264f56d92e83463d64a55f043252fafaea38040654cddd648c3caa49e5dab46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pluskal-lab/DreaMS
  tools:
  - DreaMS
  - MIST
  - MS2DeepScore
  - pluskal-lab/DreaMS
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# Fingerprint linear probing on mass spectra

## Summary

Probes frozen self-supervised mass-spectra embeddings (e.g. 1,024-dimensional DreaMS vectors) with progressively trained linear heads to recover MACCS molecular fingerprint bits, revealing at what training depth the model discovers interpretable molecular fragments.

## When to use

Use when you have pre-trained spectral embeddings (e.g. from DreaMS on GeMS HDF5 spectra) and want to test whether the representation space encodes molecular substructure without fine-tuning the encoder.

## When NOT to use

- You need production-quality structure annotation from spectra — the authors state fine-tuned DreaMS fingerprint prediction 'should be understood as a proof of concept', and probing is diagnostic, not an annotation tool
- You want to predict fingerprints from raw spectra without a pre-trained encoder — use a supervised fingerprint predictor such as MIST or MIST-CF instead
- The input is already a labeled feature table for supervised classification — a linear probe assumes an unlabeled self-supervised representation being evaluated, not a supervised modeling pipeline

## Inputs

- Frozen DreaMS model or pre-extracted 1,024-dimensional embeddings for held-out MS/MS spectra (GeMS/GeMS-A10 subset, HDF5 format from MassIVE GNPS mining)
- MACCS molecular fingerprint bit annotations for the molecules corresponding to the spectra (e.g. from 2D InChI-key-grouped structures)

## Outputs

- Per-depth linear-probe precision values for MACCS fingerprint bit recovery
- Bit-level precision table showing which molecular fragments are linearly decodable at each training depth

## How to apply

Extract 1,024-dimensional real-valued DreaMS embeddings for held-out spectra and freeze the encoder — the probe must be linear only so success reflects information already present in the self-supervised representation. Progressively train linear probes (one per probing depth during self-supervised pre-training) to recover individual MACCS molecular fingerprint bits, recording precision per bit and per depth. Rationale: if self-supervised pre-training via masked m/z-bin prediction and retention order prediction on GeMS has captured interpretable fragment structure, linearly decodable fingerprint information should increase monotonically during training. Validate by checking that linear-probing precision of MACCS bits falls in the reported 0.81–0.84 regime; embeddings that also show k=3 linear clustering accuracy near 0.865 in a 1,023-dimensional PCA subspace (grouped by 2D InChI key) further confirm structural organization. Compare against supervised baselines such as MIST fingerprint prediction only as context, since the probe deliberately uses no labels beyond fingerprint annotations.

## Related tools

- **DreaMS** (Source of the frozen 1,024-dimensional self-supervised spectral embeddings probed for MACCS bits; code and GeMS data from the pluskal-lab/DreaMS repository) — https://github.com/pluskal-lab/DreaMS
- **MIST** (Supervised fingerprint-prediction baseline used to contextualize how much structure a linear probe recovers from the self-supervised representation) — https://github.com/samgoldman97/mist
- **MS2DeepScore** (Supervised embedding baseline; DreaMS cosine similarity in embedding space outperforms it, supporting the quality of the probed representations) — https://github.com/matchms/ms2deepscore

## Examples

```
from sklearn.linear_model import LogisticRegression; probe = LogisticRegression().fit(X_train_1024d, y_maccs_bits); print(probe.score(X_test_1024d, y_maccs_bits_test))
```

## Evaluation signals

- Linear-probing precision for MACCS bits falls in the reported 0.81–0.84 regime
- Probe precision increases progressively with self-supervised training depth (progressive discovery of molecular fragments)
- Accompanying k=3 linear clustering accuracy in the 1,023-dimensional PCA subspace is near the reported 0.865 when spectra are grouped by 2D InChI key
- Zero-shot cosine similarity in the probed embedding space outperforms MS2DeepScore on molecular similarity prediction
- Probe weights are fit only on training spectra and evaluated on held-out spectra with disjoint molecules

## Limitations

- Fine-tuned DreaMS fingerprint prediction is explicitly a proof of concept, not production-ready, so probe results should not be read as an annotation method
- DreaMS was trained only on positive ionization mode, singly charged precursor ions; probe conclusions may not transfer to other MS conditions
- The method disregards MS1 isotopic patterns and adduct distributions, so fragments linked to those features cannot be probed
- Comparison benchmarks such as the MIST CANOPUS benchmark are limited in size (~8,000 spectra from 7,000 unique molecules)

## Evidence

- [abstract] Self-supervised pre-training yields fragments decodable by linear probes: "the DreaMS representations (1,024-dimensional real-valued vectors) are organized according to the structural similarity between molecules and are robust to mass spectrometry conditions"
- [abstract] Pre-training objectives create the representation being probed: "pre-trained it on our GeMS data to predict masked spectral peaks and chromatographic retention orders"
- [abstract] Fingerprints are among the supervised adaptations of the frozen encoder: "we adapted the network to the prediction of spectral similarity, molecular fingerprints, chemical properties and the identification of fluorine-containing molecules"
- [methods] Repository provides the model and data needed to extract embeddings: "# Install requirements
pip install -r requirements.txt"
