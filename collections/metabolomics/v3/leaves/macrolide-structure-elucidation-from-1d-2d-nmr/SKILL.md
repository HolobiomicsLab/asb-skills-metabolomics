---
name: macrolide-structure-elucidation-from-1d-2d-nmr
description: Use when a fractionated marine cyanobacterial (Symploca) extract contains
  cytotoxic, SMART 2.0-predicted macrolide/polyketide fractions and you need to resolve
  planar structures of swinholide-like compounds from NMR plus LC-MS2 data.
when_to_use_negative:
- Input is a single pure compound rather than a fractionated mixture — the prioritization
  workflow (cytotoxicity + SMART 2.0 + molecular networking) assumes fractions of
  a crude extract
- Only LC-MS2 data are available with no 1D NMR spectra — SMART 2.0 class predictions
  require 1D NMR input
- Target compounds are outside natural-product classes SMART 2.0 covers poorly and
  no NMR/LC-MS2 corroborating data can be acquired for cross-validation
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0154
tools:
- name: SMART 2.0
  role: CNN-based classifier run on each fraction's 1D NMR spectra to predict natural-product
    class and guide which macrolide-containing fractions to prioritize for isolation
- name: molecular networking
  role: MS2-based molecular networking used alongside SMART 2.0 and cytotoxicity to
    prioritize and interpret fractions
- name: NP-Classifier
  role: Dockerized NP Classifier server that programmatically classifies natural products
    from SMILES strings (e.g. via /classify?smiles=<>), used to assign compound classes
    during annotation
  repo: https://github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:50e4ce3a63a5a4ab833265059942ee28f20025bec709c57d47eaa58b8e16dc60
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - NP-Classifier
  - SMART 2.0
  - molecular networking
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# macrolide-structure-elucidation-from-1d-2d-nmr

## Summary

Elucidate and annotate macrolide-type natural products (e.g. swinholides, samholides, symplocolide A) from fractionated cyanobacterial extracts by combining 1D/2D NMR and LC-MS2 evidence, optionally accelerated by SMART 2.0 class predictions to prioritize which fractions to work up.

## When to use

Use when a fractionated marine cyanobacterial (Symploca) extract contains cytotoxic, SMART 2.0-predicted macrolide/polyketide fractions and you need to resolve planar structures of swinholide-like compounds from NMR plus LC-MS2 data.

## When NOT to use

- Input is a single pure compound rather than a fractionated mixture — the prioritization workflow (cytotoxicity + SMART 2.0 + molecular networking) assumes fractions of a crude extract
- Only LC-MS2 data are available with no 1D NMR spectra — SMART 2.0 class predictions require 1D NMR input
- Target compounds are outside natural-product classes SMART 2.0 covers poorly and no NMR/LC-MS2 corroborating data can be acquired for cross-validation

## Inputs

- 1D NMR spectra of each roughly fractionated Symploca extract fraction
- LC-MS2 data for the same fractions
- 2D NMR spectra (e.g. for planar structure confirmation of isolated compounds)
- MS2-based molecular network of the extract
- Cytotoxicity measurements against cancer cell lines for fraction prioritization

## Outputs

- Annotation table (e.g. symploca_annotations.csv) with one row per compound: compound name, natural-product class, supporting NMR/LC-MS2 evidence
- SMART 2.0 natural-product class predictions per fraction
- Elucidated planar structures (e.g. symplocolide A as a swinholide A / luminaolide B hybrid)

## How to apply

Roughly fractionate the environmental Symploca extract and record 1D NMR spectra and LC-MS2 data for each fraction. Run SMART 2.0 on each fraction's 1D NMR data to obtain automated natural-product class predictions, then combine these predictions with MS2-based molecular networking and cancer cell cytotoxicity to decide which fractions to prioritize for isolation. For each prioritized compound, cross-validate the SMART 2.0 classification against 1D/2D NMR and LC-MS2 evidence; for symplocolide A this confirmed a planar structure that is a structural hybrid between swinholide A and luminaolide B. Compile a structured annotation table (one row per compound: name, natural-product class, supporting NMR/LC-MS2 evidence) covering swinholide A, samholides A–I, new derivatives, and symplocolide A.

## Related tools

- **SMART 2.0** (CNN-based classifier run on each fraction's 1D NMR spectra to predict natural-product class and guide which macrolide-containing fractions to prioritize for isolation)
- **molecular networking** (MS2-based molecular networking used alongside SMART 2.0 and cytotoxicity to prioritize and interpret fractions)
- **NP-Classifier** (Dockerized NP Classifier server that programmatically classifies natural products from SMILES strings (e.g. via /classify?smiles=<>), used to assign compound classes during annotation) — https://github.com/mwang87/NP-Classifier

## Examples

```
curl 'http://localhost/classify?smiles=<symplocolide_A_smiles>'  # after: docker network create nginx-net && make server-compose
```

## Evaluation signals

- symploca_annotations.csv exists and contains entries for swinholide A, samholides, and symplocolide A
- Annotation table matches the annotation set reported in the article (output_matches_reference)
- Symplocolide A's planar structure is supported by 1D/2D NMR and LC-MS2 analysis and is consistent with a swinholide A / luminaolide B hybrid
- SMART 2.0 class predictions are cross-validated against the 1D/2D NMR and LC-MS2 evidence for each annotated compound
- Model server sanity check: NP Classifier input layer names are 'input_2048' and 'input_4096' and output layer name is 'output'

## Limitations

- SMART 2.0 predictions are machine-learning class calls and must be cross-validated against 1D/2D NMR and LC-MS2 evidence rather than trusted alone
- If the NP Classifier model input names change, code must be updated — the server expects inputs 'input_2048' and 'input_4096' with output layer 'output'
- Model conversion requires a pinned dependency: tensorflow 2.3.0 to convert the keras models into HDF5 TF2 models
- No changelog is available for the NP Classifier repository

## Evidence
