---
name: peptide-binding-affinity-prediction-context-analysis
description: Use when reproducing or interpreting benchmark results on PSM rescoring
  features for immunopeptidomics, e.g. judging whether binding-affinity, ΔRT, or spectral
  features from tools like inSPIRE, DeepRescore, or MHCquant actually improve identifications.
when_to_use_negative:
- Input is already a scored feature table with rescoring features computed — this
  skill analyzes reported benchmark findings, not feature generation itself
- The task is configuring or running an MS²Rescore pipeline (modification mappings,
  spectrum linking, FDR filtering) rather than interpreting which predicted properties
  are informative
- The source article reports no empirical benchmark results with named tools, so no
  finding/tool/condition records can be extracted
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: MS2Rescore
  role: Modular platform for AI-assisted rescoring that integrates predicted peptidoform
    features (MS²PIP, DeepLC) used to reproduce and contextualize benchmark findings
  repo: https://github.com/compomics/ms2rescore
- name: Mumble
  role: PSM generator proposing Unimod modifications that explain unresolved precursor
    mass shifts before feature generation and rescoring
  repo: https://github.com/compomics/mumble
- name: ristretto
  role: Lean Percolator-style semi-supervised rescoring engine (svm or lda) that trains
    a classifier to separate correct from incorrect PSMs and reports q-values and
    PEPs
  repo: https://github.com/CompOmics/ristretto
- name: NetMHCpan
  role: Binding-affinity predictor whose outputs form predicted peptidoform features
    for rescoring (e.g. inSPIRE's binding-affinity features)
- name: Percolator
  role: Semi-supervised rescoring reference tool; benchmarked for the numProt bias
    finding
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:bf1ce8508974e980c56f4064cb5e655efe710d542857ce5f847f90f92788b033
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compomics/ms2rescore
  repo_url: https://github.com/compomics/ms2rescore
  tools:
  - DeepRescore
  - MHCquant
  - MS2Rescore
  - MSBooster
  - Mumble
  - NetMHCpan
  - Percolator
  - inSPIRE
  - ristretto
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Peptide-binding-affinity-prediction context analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Structured analysis of how predicted peptidoform properties — MHC binding affinity, retention time (ΔRT), fragment ion intensities, and collisional cross section — contribute as features in machine-learning PSM rescoring for immunopeptidomics, benchmarking which tool/feature/condition combinations yield reported identification gains.

## When to use

Use when reproducing or interpreting benchmark results on PSM rescoring features for immunopeptidomics, e.g. judging whether binding-affinity, ΔRT, or spectral features from tools like inSPIRE, DeepRescore, or MHCquant actually improve identifications.

## When NOT to use

- Input is already a scored feature table with rescoring features computed — this skill analyzes reported benchmark findings, not feature generation itself
- The task is configuring or running an MS²Rescore pipeline (modification mappings, spectrum linking, FDR filtering) rather than interpreting which predicted properties are informative
- The source article reports no empirical benchmark results with named tools, so no finding/tool/condition records can be extracted

## Inputs

- Full text of a PSM-rescoring review article (DOI 10.1002/pmic.202300336, 'Machine learning-based peptide-spectrum match rescoring opens up the immunopeptidome')
- PSM identification files (e.g. mzIdentML or TSV) from search engines, optionally with decoy labels for FDR control
- Predicted peptidoform property values: MHC binding affinity (e.g. NetMHCpan), retention time/ΔRT, fragment ion intensities (e.g. MS²PIP/Prosit), collisional cross section

## Outputs

- findings_table.json containing exactly 7 records with non-empty 'finding', 'tool', and 'condition' fields mapping each benchmark result to its tool and experimental condition
- Structured tool-condition-feature attribution explaining contradictory findings (e.g. MHCquant ΔRT ineffectiveness vs DeepRescore ΔRT dominance)

## How to apply

Load the full text of the review (DOI 10.1002/pmic.202300336) and locate the empirical benchmark results: DeepRescore's ΔRT plus spectral-angle dominance in DDA immunopeptidomics, inSPIRE's binding-affinity/spectral-feature concert, MHCquant's ΔRT ineffectiveness, MSBooster's marginal ion-mobility benefit, numProt bias in Percolator, Prosit finetuned for timsTOF, and transformer-with-pretraining outperforming Prosit's RNN. Parse each result into a record with 'finding', 'tool', and 'condition' fields, noting that MHCquant's negative ΔRT result is attributed to its outdated RTPredict predictor (published more than 15 years ago) whereas DeepRescore found ΔRT consistently among the highest-weight features — i.e., highly accurate predicted peptide properties are required for informative rescoring features. Assemble the records into a single table serialized to findings_table.json, then validate that the file parses as JSON, contains exactly 7 records with non-empty fields, and that each tool-condition mapping is checked by expert review against the article text.

## Related tools

- **MS2Rescore** (Modular platform for AI-assisted rescoring that integrates predicted peptidoform features (MS²PIP, DeepLC) used to reproduce and contextualize benchmark findings) — https://github.com/compomics/ms2rescore
- **Mumble** (PSM generator proposing Unimod modifications that explain unresolved precursor mass shifts before feature generation and rescoring) — https://github.com/compomics/mumble
- **ristretto** (Lean Percolator-style semi-supervised rescoring engine (svm or lda) that trains a classifier to separate correct from incorrect PSMs and reports q-values and PEPs) — https://github.com/CompOmics/ristretto
- **NetMHCpan** (Binding-affinity predictor whose outputs form predicted peptidoform features for rescoring (e.g. inSPIRE's binding-affinity features))
- **Percolator** (Semi-supervised rescoring reference tool; benchmarked for the numProt bias finding)

## Examples

```
python -c "import json; d=json.load(open('findings_table.json')); assert len(d)==7 and all(all(r[k] for k in ('finding','tool','condition')) for r in d)"
```

## Evaluation signals

- findings_table.json exists, parses as valid JSON, and contains exactly 7 records
- Every record has non-empty 'finding', 'tool', and 'condition' fields
- The tool-condition mapping for each of the 7 results is verified by expert review against the article text
- Contradictory ΔRT conclusions (MHCquant vs DeepRescore) are attributed to predictor quality (outdated RTPredict vs modern predictors), not to the feature being intrinsically uninformative
- Each benchmarked tool (DeepRescore, inSPIRE, MHCquant, MSBooster, Prosit, Percolator) appears in exactly one tool-condition mapping

## Limitations

- The reviewed discussion section is a changelog rather than scientific discussion, so no evaluation metrics or validation data are described there; benchmark conclusions must come from the empirical-results sections
- Percolator is referenced as an external dependency without a specified version, complicating exact reproduction of findings such as the numProt bias
- Feature informativeness is predictor-dependent: MHCquant's use of the outdated RTPredict retention time predictor (published more than 15 years ago) yielded inaccurate predictions, so negative feature results may reflect model quality rather than feature value

## Evidence

- [abstract] Predicted properties as rescoring features: "features based on predicted peptidoform properties, including fragment ion intensities, retention time, and collisional cross section, have been used to improve the accuracy and sensitivity of"
- [abstract] PSM rescoring as ML solution: "Peptide-spectrum match rescoring has emerged as a machine learning-based solution to address challenges in mass spectrometry-based immunopeptidomics data analysis"
- [abstract] Immunopeptide identification difficulty: "identifying immunopeptides remains challenging due to their non-tryptic nature, which results in distinct spectral characteristics"
- [readme] MS2Rescore rescoring with predictors: [evidence span withheld — anchor status: external_doc]
- [readme] ristretto classifier outputs: [evidence span withheld — anchor status: external_doc]
