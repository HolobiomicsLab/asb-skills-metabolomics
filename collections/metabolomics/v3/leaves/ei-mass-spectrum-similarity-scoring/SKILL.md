---
name: ei-mass-spectrum-similarity-scoring
description: Use when a GC/EI-MS analyte's spectrum does not match any known reference
  library and candidate identities must be ranked against predicted derivative spectra.
when_to_use_negative:
- The analyte's spectrum already matches an established reference spectrum library
  (NIST, Wiley, SWGDRUG), so conventional library matching suffices.
- The chemical structure of the NPS is already known, making direct EI spectrum prediction
  (e.g., NEIMS alone) more appropriate than integrated spectrum+fingerprint scoring.
- Input data are retention-time or high-resolution MS/MS fragmentation data rather
  than 70 eV EI spectra, which PS2MS's SMSF metric is not designed to handle.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: PS2MS
  role: Main pipeline that performs enumeration, spectrum/fingerprint prediction,
    and integrated similarity scoring (SMSF) between the unknown analyte and the synthetic
    NPS database.
  repo: https://github.com/jhhung/PS2MS
- name: DeepEI
  role: Converts the analyte's mass spectrum into its corresponding molecular fingerprint
    without explicit structure information, enabling fingerprint-based similarity.
  repo: https://github.com/jhhung/PS2MS
- name: NEIMS
  role: Predicts the mass spectrum of all derivatives in the synthetic NPS database,
    providing the reference spectra against which the analyte is scored.
- name: rdkit
  role: Required for building the enumeration step and running the detection step
    of the pipeline (mass spectra and fingerprint comparisons).
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:18a3fefe0f1f3707cc05347b1a8543712d8bb3887f4c8a92ad1c6d56ddcfb55b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - DeepEI
  - NEIMS
  - PS2MS
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# EI-mass-spectrum-similarity-scoring

## Summary

Computes integrated similarity scores (SMSF) between an unknown GC/EI-MS analyte and predicted mass spectra plus chemical fingerprints of enumerated derivative compounds, ranking candidates to identify novel psychoactive substances lacking reference spectra.

## When to use

Use when a GC/EI-MS analyte's spectrum does not match any known reference library and candidate identities must be ranked against predicted derivative spectra.

## When NOT to use

- The analyte's spectrum already matches an established reference spectrum library (NIST, Wiley, SWGDRUG), so conventional library matching suffices.
- The chemical structure of the NPS is already known, making direct EI spectrum prediction (e.g., NEIMS alone) more appropriate than integrated spectrum+fingerprint scoring.
- Input data are retention-time or high-resolution MS/MS fragmentation data rather than 70 eV EI spectra, which PS2MS's SMSF metric is not designed to handle.

## Inputs

- EI mass spectrum of the unknown analyte (GC/EI-MS, 70 eV)
- NEIMS-predicted EI mass spectra of enumerated derivative compounds in the synthetic NPS database
- Molecular fingerprints predicted by DeepEI for the unknown analyte
- Chemical fingerprints (generated via DeepEI) for compounds in the synthetic NPS database

## Outputs

- SMSF similarity scores for every analyte–derivative pair
- Ranked list of the hundred most similar compounds per analyte (with chemical structure formulas)
- Per-substance rank tables and aggregate top-20 recovery proportion

## How to apply

First, run DeepEI on the analyte's EI mass spectrum to infer its molecular fingerprint without knowing the chemical structure. Next, compute pairwise similarities between the analyte spectrum and each NEIMS-predicted derivative spectrum: cosine similarity and Jaccard similarity (overlap of top-intensity peaks), both with a ±1 Da mass tolerance used by PS2MS. Then combine mass-spectrum similarity with fingerprint similarity as a weighted-sum SMSF score for every analyte–derivative pair. Finally, rank all database derivatives per substance by their SMSF score and report the top candidates; the article reports over 85% of SWGDRUG test substances ranked within the top 20 predictions.

## Related tools

- **PS2MS** (Main pipeline that performs enumeration, spectrum/fingerprint prediction, and integrated similarity scoring (SMSF) between the unknown analyte and the synthetic NPS database.) — https://github.com/jhhung/PS2MS
- **DeepEI** (Converts the analyte's mass spectrum into its corresponding molecular fingerprint without explicit structure information, enabling fingerprint-based similarity.) — https://github.com/jhhung/PS2MS
- **NEIMS** (Predicts the mass spectrum of all derivatives in the synthetic NPS database, providing the reference spectra against which the analyte is scored.)
- **rdkit** (Required for building the enumeration step and running the detection step of the pipeline (mass spectra and fingerprint comparisons).) — https://www.rdkit.org/docs/Install.html

## Examples

```
python drug-detection/run.py --analyte spectrum.msp --database synthetic_nps.msp --top 100
```

## Evaluation signals

- Verify that the computed top-20 recovery proportion on the SWGDRUG test set reproduces the reported value of over 85%.
- Check that incorporating Jaccard similarity raises the top-1 correct-prediction proportion from below 40% to more than 50%.
- Confirm the ranked output list contains the hundred most similar compounds ranked by SMSF similarity score, as specified in the drug-detection step.
- Validate that a ±1 Da mass tolerance is applied when computing Jaccard similarity of top-intensity peaks.

## Limitations

- The system was validated primarily on cathinone-like derivatives; performance on other NPS structural classes is not demonstrated here.
- NEIMS was initially trained solely on NIST spectra, so predicted derivative spectra may be less reliable for chemically distant scaffolds.
- Anticipated incorporation of additional features is noted as future work to enhance prediction accuracy.

## Evidence

- [methods] SMSF metric combines mass-spectrum and fingerprint similarity: "The proportion of correct prediction appearing at top 1 increased from below 40% to more than 50% after incorporating Jaccard similarity."
- [methods] Top-20 recovery rate reported: "over 90% and 85% of the test substances, respectively, ranked within the top 20 predictions"
- [readme] PS2MS pipeline overview from README: "PS<sup>2</sup>MS calculates the integrated similarity scores(SMSF) between the unknown analyte and the derivatives from synthetic database and yields a list of potential NPS identities for the"
- [intro] DeepEI role in fingerprint inference: "PS2MS employs the DeepEI30 deep learning model to convert the analyte’s mass spectrum into its corresponding fingerprint"
- [methods] NEIMS role in derivative spectra prediction: "we employed NEIMS to predict the mass spectrum of all derivatives within our synthetic NPS database."
