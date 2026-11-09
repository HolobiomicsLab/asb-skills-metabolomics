---
name: natural-product-isolation-prioritization
description: Use when a crude natural product extract (e.g., marine cyanobacterial
  LC-MS2 data) must be fractionated and you need to decide which fractions to pursue
  based on MS2 molecular networking, cytotoxicity, and NMR mixture analysis signals.
when_to_use_negative:
- When the extract is already resolved into a purified compound — this skill targets
  prioritization of rough fractionation of crude mixtures
- When no MS2 or cytotoxicity data can be acquired, since prioritization relies on
  the combination of MS2-based molecular networking, cancer cell cytotoxicity, and
  SMART 2.0
- When the goal is purely computational dereplication of a spectral library without
  any follow-up isolation work
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: molecular networking
  role: MS2-based molecular networking used to visualize and prioritize fractions
    via spectral library matching and MS2 cosine-similarity edge scores
- name: SMART 2.0
  role: NMR-based machine learning tool applied to mixture analysis to annotate chemical
    classes and guide fractionation without prior isolation
- name: NP-Classifier
  role: Programmatic chemical-class classification of candidate structures from SMILES
    strings via the /classify API
  repo: https://github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:9f0b767d8587ab4b3fae2d45a7d66325fa2a40dd89df36b2ac5aebdd24e99093
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - NP-Classifier
  - SMART 2.0
  - molecular networking
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# natural-product-isolation-prioritization

## Summary

Prioritizes and guides fractionation of a crude natural product extract by combining MS2-based molecular networking (GNPS cosine-similarity edges), cancer cell cytotoxicity screening, and NMR-based SMART 2.0, so that isolation effort is concentrated on chemically and biologically interesting fractions. Applied to an environmental Symploca extract this workflow yielded symplocolide A and annotations of swinholide A, samholides A–I, and new derivatives.

## When to use

Use when a crude natural product extract (e.g., marine cyanobacterial LC-MS2 data) must be fractionated and you need to decide which fractions to pursue based on MS2 molecular networking, cytotoxicity, and NMR mixture analysis signals.

## When NOT to use

- When the extract is already resolved into a purified compound — this skill targets prioritization of rough fractionation of crude mixtures
- When no MS2 or cytotoxicity data can be acquired, since prioritization relies on the combination of MS2-based molecular networking, cancer cell cytotoxicity, and SMART 2.0
- When the goal is purely computational dereplication of a spectral library without any follow-up isolation work

## Inputs

- LC-MS2 peak list of the crude Symploca extract
- Rough fractions of the environmental extract for cytotoxicity testing
- 1D/2D NMR spectra of crude mixtures and isolated compounds (input to SMART 2.0)
- SMILES strings of candidate structures (for NP Classifier annotation)

## Outputs

- Molecular networking nodes/edges tables (molecular_network.tsv) with MS2 cosine-similarity edge scores
- Prioritized shortlist of fractions for isolation
- Isolated and structurally identified compounds (e.g., symplocolide A by 1D/2D NMR and LC-MS2)
- Annotations of known analogs (swinholide A, samholides A–I) and new derivatives
- NP Classifier chemical-class assignments via /classify?smiles=<>

## How to apply

Start by roughly fractionating the environmental extract, then acquire LC-MS2 data and load the peak list for molecular networking: run spectral library matching and propagate edge scores via the MS2 cosine similarity used in GNPS molecular networking, exporting the network as nodes/edges tables (molecular_network.tsv). In parallel, run cancer cell cytotoxicity assays on fractions and apply SMART 2.0, the NMR-based machine learning tool, to the crude mixture to annotate chemical classes without prior isolation. Use the intersection of network nodes matching known bioactive chemotypes (e.g., swinholide/samholide macrolides), cytotoxic fractions, and SMART 2.0 class predictions to prioritize which fractions to scale up and purify. Follow prioritized fractions through isolation (e.g., yielding symplocolide A) and confirm planar structures by 1D/2D NMR and LC-MS2 analysis. Optionally classify isolated/predicted structures programmatically via the NP Classifier API (/classify?smiles=<>).

## Related tools

- **molecular networking** (MS2-based molecular networking used to visualize and prioritize fractions via spectral library matching and MS2 cosine-similarity edge scores)
- **SMART 2.0** (NMR-based machine learning tool applied to mixture analysis to annotate chemical classes and guide fractionation without prior isolation)
- **NP-Classifier** (Programmatic chemical-class classification of candidate structures from SMILES strings via the /classify API) — https://github.com/mwang87/NP-Classifier

## Examples

```
cd Classifier/models_folder/models && sh ./get_models.sh && make server-compose
```

## Evaluation signals

- molecular_network.tsv exports both nodes and edges tables with propagated MS2 cosine-similarity edge scores
- Prioritized fractions show concordance between cytotoxicity, SMART 2.0 class predictions, and molecular networking clusters
- Isolated compounds are confirmed by 1D/2D NMR and LC-MS2 analysis (e.g., symplocolide A confirmed as a structural hybrid between swinholide A and luminaolide B)
- Known analogs (swinholide A, samholides A–I) are recovered as annotations, indicating the network connects related macrolides
- NP Classifier /classify endpoint returns class predictions for input SMILES with input layers named 'input_2048' and 'input_4096' and output layer 'output'

## Limitations

- SMART 2.0 requires tensorflow 2.3.0 to convert keras models into HDF5 TF2 models, and the dockerized server depends on specific model input/output layer names ('input_2048', 'input_4096', 'output') — if these change, code must be updated
- Prioritization is guided by a specific bioassay (cancer cell cytotoxicity); fractions without cytotoxicity may be deprioritized even if chemically interesting
- The workflow was demonstrated on an environmental Symploca (marine cyanobacterium) extract; applicability to other extract types is not established in this context

## Evidence

- [methods] networking workflow: [evidence span withheld — anchor status: not_found]
