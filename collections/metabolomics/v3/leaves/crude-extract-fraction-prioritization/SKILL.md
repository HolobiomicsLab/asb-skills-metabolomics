---
name: crude-extract-fraction-prioritization
description: Use when a crude natural-product extract has been roughly fractionated
  and you need to decide which fractions to interpret or isolate from, guided by cytotoxicity,
  NMR-based SMART 2.0, and MS2-based molecular networking.
when_to_use_negative:
- Input is already a purified compound whose structure is being elucidated from scratch
  — this skill is for prioritizing and interpreting fractions of a crude, roughly
  fractionated extract, not for single-compound de novo structure determination.
- The extract has not been fractionated (no per-fraction NMR/LC-MS2 data to prioritize)
  — apply fractionation first before using this skill.
- The goal is annotation of already-known compounds by SMILES lookup via NP Classifier
  API — use the classify endpoint instead of the extract-guided prioritization workflow.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3336
tools:
- name: SMART 2.0
  role: NMR-based machine learning tool run on each fraction's 1D NMR data to obtain
    automated natural-product class predictions used to prioritize and guide fraction
    interpretation
- name: molecular networking
  role: MS2-based molecular networking of per-fraction LC-MS2 data used alongside
    SMART 2.0 and cytotoxicity to guide fraction prioritization
- name: NP-Classifier
  role: Dockerized NP Classifier server (mwang87/NP-Classifier) that deploys the SMART
    2.0 models for natural-product classification from NMR-derived input features
  repo: https://github.com/mwang87/NP-Classifier
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jacs.9b13786
    title: A Convolutional Neural Network-Based Approach for the Rapid Annotation
      of Molecularly Diverse Natural Products
  source_content_hash: sha256:9b8273a740da8c0fb504d683f3009d9f361a551e57846ec7d016462765be7a31
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mwang87/NP-Classifier
  tools:
  - NP-Classifier
  - SMART 2.0
  - molecular networking
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# crude-extract-fraction-prioritization

## Summary

A workflow skill for prioritizing and guiding interpretation of roughly fractionated crude natural-product extracts by combining cancer cell cytotoxicity, NMR-based SMART 2.0 class predictions, and MS2-based molecular networking, exemplified on an environmental Symploca extract to rapidly isolate symplocolide A and annotate swinholide A, samholides A-I, and new derivatives.

## When to use

Use when a crude natural-product extract has been roughly fractionated and you need to decide which fractions to interpret or isolate from, guided by cytotoxicity, NMR-based SMART 2.0, and MS2-based molecular networking.

## When NOT to use

- Input is already a purified compound whose structure is being elucidated from scratch — this skill is for prioritizing and interpreting fractions of a crude, roughly fractionated extract, not for single-compound de novo structure determination.
- The extract has not been fractionated (no per-fraction NMR/LC-MS2 data to prioritize) — apply fractionation first before using this skill.
- The goal is annotation of already-known compounds by SMILES lookup via NP Classifier API — use the classify endpoint instead of the extract-guided prioritization workflow.

## Inputs

- Roughly fractionated crude extract (per-fraction samples)
- Per-fraction 1D NMR spectra
- Per-fraction LC-MS2 data (for molecular networking)
- Cancer cell cytotoxicity assay results per fraction

## Outputs

- SMART 2.0 natural-product class predictions per fraction
- MS2 molecular-networking maps per extract
- Prioritized list of fractions for isolation
- Structured annotation table (one row per compound: name, natural-product class, supporting NMR/LC-MS2 evidence)

## How to apply

Roughly fractionate the crude extract, then score each fraction for cancer cell cytotoxicity and record 1D NMR and LC-MS2 data per fraction. Run SMART 2.0 (the 'Small Molecule Accurate Recognition Technology' convolutional neural network) on each fraction's 1D NMR spectra to obtain automated natural-product class predictions, and run MS2-based molecular networking on the LC-MS2 data to see which molecular families correlate with cytotoxicity and predicted classes. Prioritize fractions whose cytotoxicity, NMR class prediction, and molecular-networking evidence converge (e.g. fractions predicting macrolide-like classes that network with known swinholide-like nodes). Interpret prioritized fractions with 1D/2D NMR and LC-MS2 to isolate and annotate compounds; in the source study this converged on a structural hybrid between swinholide A and luminaolide B, named symplocolide A, plus swinholide A, samholides A-I, and several new derivatives. For each annotated compound, cross-validate the SMART 2.0 class call against the manual 1D/2D NMR and LC-MS2 structural evidence before recording it in an annotation table.

## Related tools

- **SMART 2.0** (NMR-based machine learning tool run on each fraction's 1D NMR data to obtain automated natural-product class predictions used to prioritize and guide fraction interpretation)
- **molecular networking** (MS2-based molecular networking of per-fraction LC-MS2 data used alongside SMART 2.0 and cytotoxicity to guide fraction prioritization)
- **NP-Classifier** (Dockerized NP Classifier server (mwang87/NP-Classifier) that deploys the SMART 2.0 models for natural-product classification from NMR-derived input features) — https://github.com/mwang87/NP-Classifier

## Examples

```
docker network create nginx-net && make server-compose # then curl 'http://localhost/classify?smiles=<compound_smiles>' to classify candidate structures in prioritized fractions
```

## Evaluation signals

- symploca_annotations.csv exists and contains entries for swinholide A, samholides, and symplocolide A
- SMART 2.0 class predictions per fraction are recorded and cross-validated against 1D/2D NMR and LC-MS2 evidence for each annotated compound
- Planar structure of symplocolide A is confirmed as a structural hybrid between swinholide A and luminaolide B by 1D/2D NMR and LC-MS2 analysis
- Annotation table matches the compound set reported in the article (swinholide A, samholides A-I, several new derivatives, symplocolide A)

## Limitations

- SMART 2.0 predictions must be cross-validated against manual 1D/2D NMR and LC-MS2 evidence; automated class calls alone are not sufficient for structural confirmation.
- Prioritization depends on having per-fraction cytotoxicity data; fractions lacking assay data cannot be ranked by this combined guidance scheme.
- The demonstrated workflow was validated on an environmental Symploca (marine cyanobacterium) extract; transfer to other extract types is plausible but not demonstrated here.

## Evidence

- [abstract] first application of the novel NMR-based machine learning tool ‘Small Molecule Accurate Recognition Technology’ (SMART 2.0) for mixture analysis: "first application of the novel NMR-based machine learning tool ‘Small Molecule Accurate Recognition Technology’ (SMART 2.0) for mixture analysis"
- [abstract] prioritized and guided by cancer cell cytotoxicity, NMR-based SMART 2.0, and MS 2 -based molecular networking: "prioritized and guided by cancer cell cytotoxicity, NMR-based SMART 2.0, and MS 2 -based molecular networking"
- [abstract] The planar structure of symplocolide A was confirmed to be a structural hybrid between swinholide A and luminaolide B by 1D/2D NMR and LC-MS 2 analysis: "The planar structure of symplocolide A was confirmed to be a structural hybrid between swinholide A and luminaolide B by 1D/2D NMR and LC-MS 2 analysis"
- [readme] Input layers' names should be "input_2048" and "input_4096": "Input layers' names should be "input_2048" and "input_4096""
