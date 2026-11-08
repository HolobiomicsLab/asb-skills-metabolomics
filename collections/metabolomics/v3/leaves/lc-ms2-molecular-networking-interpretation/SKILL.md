---
name: lc-ms2-molecular-networking-interpretation
description: Use when LC-MS/MS data from fractionated crude natural-product extracts
  (e.g. marine cyanobacterial Symploca fractions) must be clustered by MS2 similarity
  to prioritize which fractions to isolate and annotate.
when_to_use_negative:
- When the goal is standalone NP class prediction from SMILES strings — use the NP
  Classifier API instead of molecular networking
- When only 1D NMR spectra are available and no MS2 fragmentation data exist to build
  the network
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: molecular networking
  role: MS2-based clustering of LC-MS/MS spectra from the fractionated Symploca extract
    to guide fraction prioritization and compound annotation
- name: SMART 2.0
  role: Complementary NMR-based convolutional neural network class predictions used
    jointly with molecular networking to prioritize and annotate fractions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:61297add1a01ce644d34fd376ba28a01815df723530469088cde79718fdb022a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SMART 2.0
  - molecular networking
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lc-ms2-molecular-networking-interpretation

## Summary

Interpret MS2-based molecular networks built from LC-MS/MS data of fractionated natural-product extracts to cluster related metabolites and prioritize fractions for isolation and annotation, in combination with NMR-based SMART 2.0 predictions and cytotoxicity data.

## When to use

Use when LC-MS/MS data from fractionated crude natural-product extracts (e.g. marine cyanobacterial Symploca fractions) must be clustered by MS2 similarity to prioritize which fractions to isolate and annotate.

## When NOT to use

- When the goal is standalone NP class prediction from SMILES strings — use the NP Classifier API instead of molecular networking
- When only 1D NMR spectra are available and no MS2 fragmentation data exist to build the network

## Inputs

- LC-MS/MS (LC-MS2) spectra of roughly fractionated environmental Symploca extract
- 1D NMR spectra per fraction (for SMART 2.0 class predictions used alongside the network)
- Cancer cell cytotoxicity measurements per fraction (for prioritization)

## Outputs

- MS2-based molecular network linking related metabolites across fractions
- Prioritized list of fractions for isolation
- Structured annotation table (compound name, natural-product class, supporting NMR/LC-MS2 evidence), e.g. symploca_annotations.csv

## How to apply

Start from roughly fractionated extract LC-MS2 data and construct an MS2-based molecular network so that metabolites with related fragmentation patterns cluster together, revealing families such as swinholide-like macrolides across fractions. Combine the network interpretation with SMART 2.0 class predictions from each fraction's 1D NMR data and cancer cell cytotoxicity readouts to decide which fractions to prioritize for isolation. For each node of interest, cross-validate the network-based annotation against 1D/2D NMR and LC-MS2 evidence, as was done to confirm symplocolide A as a structural hybrid between swinholide A and luminaolide B. Compile the resulting annotations into a structured table (one row per compound: name, natural-product class, supporting NMR/LC-MS2 evidence) covering known compounds, new derivatives, and novel hybrids.

## Related tools

- **molecular networking** (MS2-based clustering of LC-MS/MS spectra from the fractionated Symploca extract to guide fraction prioritization and compound annotation)
- **SMART 2.0** (Complementary NMR-based convolutional neural network class predictions used jointly with molecular networking to prioritize and annotate fractions)

## Evaluation signals

- Output annotation table contains entries for swinholide A, samholides A–I, and symplocolide A
- Annotations match the reported annotation set from the article (output_matches_reference against symploca_annotations.csv)
- Each annotated compound row records natural-product class plus supporting 1D/2D NMR and LC-MS2 evidence
- Symplocolide A's planar structure is corroborated by 1D/2D NMR and LC-MS2 as a swinholide A / luminaolide B hybrid

## Limitations

- Molecular networking alone was insufficient — fraction prioritization required combining it with cytotoxicity and SMART 2.0 NMR predictions
- Annotations from the network remain planar-level and need 1D/2D NMR and LC-MS2 cross-validation for structural confirmation

## Evidence

- [full_text] Task workflow description: [evidence span withheld — anchor status: not_found]
