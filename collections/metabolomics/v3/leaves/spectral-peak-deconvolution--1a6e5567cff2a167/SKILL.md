---
name: spectral-peak-deconvolution
description: Use when LC-MS2 peak lists from crude natural product extracts (e.g.
  marine cyanobacteria) must be deconvoluted into related spectral families to prioritize
  which fractions to isolate, or to annotate known metabolites like swinholides before
  committing to purification.
when_to_use_negative:
- Input is a purified single compound whose structure still requires full 1D/2D NMR
  and LC-MS2 confirmation — networking alone cannot establish planar structure (symplocolide
  A required 1D/2D NMR and LC-MS2).
- The goal is compound class prediction from a known SMILES string — use NP Classifier's
  /classify API instead of MS2 networking.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: molecular networking
  role: MS2 cosine-similarity clustering of LC-MS2 spectra to deconvolute the extract
    into molecular families
- name: SMART 2.0
  role: NMR-based deep learning mixture analysis used jointly with networking to prioritize
    fractionation
- name: NP Classifier
  role: Programmatic chemical-class annotation of identified structures from SMILES
    via the /classify API
  repo: https://github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:140baf9e79dbc360ae5e1e3ecdfe2958ae7b77c3357e8d62524e2def3f243f8d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - NP Classifier
  - SMART 2.0
  - molecular networking
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spectral-peak-deconvolution

## Summary

Deconvolution and prioritization of LC-MS2 spectra from crude natural product extracts using MS2-based (GNPS) molecular networking, where spectral library matching and MS2 cosine similarity edge scores group related ions to guide which fractions to isolate. In this paper it was combined with cancer cell cytotoxicity and NMR-based SMART 2.0 to prioritize fractionation of a marine cyanobacterial Symploca extract.

## When to use

Use when LC-MS2 peak lists from crude natural product extracts (e.g. marine cyanobacteria) must be deconvoluted into related spectral families to prioritize which fractions to isolate, or to annotate known metabolites like swinholides before committing to purification.

## When NOT to use

- Input is a purified single compound whose structure still requires full 1D/2D NMR and LC-MS2 confirmation — networking alone cannot establish planar structure (symplocolide A required 1D/2D NMR and LC-MS2).
- The goal is compound class prediction from a known SMILES string — use NP Classifier's /classify API instead of MS2 networking.

## Inputs

- LC-MS2 peak list of the crude or roughly fractionated natural product extract
- MS2 spectral library for library matching
- Cytotoxicity assay data for prioritization (optional, used jointly here)

## Outputs

- Molecular network nodes/edges tables exported as molecular_network.tsv
- Library-matched annotations of known metabolites (e.g. swinholide A, samholides)
- Prioritized fraction list for isolation guided by cytotoxicity and SMART 2.0

## How to apply

Load the LC-MS2 peak list of the crude extract (here, the environmental Symploca extract after rough fractionation) and run spectral library matching to annotate known ions. Build a molecular network in the GNPS style, propagating edge scores via MS2 cosine similarity so related molecules (e.g. swinholide A and samholide-like macrolides) cluster as connected nodes. Export the network as nodes/edges tables (molecular_network.tsv) and read clusters together with cancer cell cytotoxicity and SMART 2.0 NMR mixture analysis to decide which fractions to pursue. Rationale: networking collapses hundreds of minor ions into interpretable families, enabling rapid identification of symplocolide A and annotation of swinholide A, samholides A–I, and several new derivatives without exhaustive isolation.

## Related tools

- **molecular networking** (MS2 cosine-similarity clustering of LC-MS2 spectra to deconvolute the extract into molecular families)
- **SMART 2.0** (NMR-based deep learning mixture analysis used jointly with networking to prioritize fractionation)
- **NP Classifier** (Programmatic chemical-class annotation of identified structures from SMILES via the /classify API) — https://github.com/mwang87/NP-Classifier

## Examples

```
curl "http://localhost/classify?smiles=<SMILES>"
```

## Evaluation signals

- Exported molecular_network.tsv contains both node and edge tables, with edges scored by MS2 cosine similarity
- Known swinholide-family members (swinholide A, samholides) are annotated by spectral library matching within the expected cluster
- Prioritized fractions yield isolates whose structures are confirmed by 1D/2D NMR and LC-MS2 (as done for symplocolide A, the swinholide A / luminaolide B hybrid)
- If deploying NP Classifier locally, the served model reports input layer names 'input_2048' and 'input_4096' and output layer name 'output' via /model/metadata

## Limitations

- Molecular networking prioritizes fractionation but does not replace NMR confirmation; planar structures must be validated by 1D/2D NMR and LC-MS2.
- Deploying NP Classifier locally requires tensorflow version 2.3.0 to convert the keras models into HDF5 TF2 models, and code changes are needed if model input layer names change.
- The extract must be at least roughly fractionated before networking-driven prioritization is meaningful; extremely complex unfractionated mixtures may not resolve.

## Evidence
