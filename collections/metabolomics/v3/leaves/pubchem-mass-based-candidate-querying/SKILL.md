---
name: pubchem-mass-based-candidate-querying
description: Use when an unannotated IM-MS feature (e.g. [M-H]- at m/z 155.0093) has
  an experimentally measured CCS and you need all plausible PubChem structures within
  a 10 ppm exact-mass window to filter by predicted ΔCCS.
when_to_use_negative:
- The analyte is a dimer, less common adduct, or in-source fragment (e.g. [2M+H]+,
  [2M-H]-) — CCSP 2.0's training sets do not support these species
- No experimental CCS measurement exists for the feature, so the ΔCCS filtering rationale
  that motivates the candidate query cannot be applied
- Very large-scale screening through the Google Colab route — Colab limits continuous
  operation to ~12 hours and disconnects after ~20 minutes idle; use the local Jupyter
  Lab notebook instead
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: Biopython (Entrez)
  role: Issues the PubChem exact-mass candidate queries via the Entrez Global Query
    Cross-Database Search System
- name: PubChemPy
  role: Retrieves each candidate CID's isomeric SMILES from PubChem
- name: RDKit
  role: Encodes MOL files from the retrieved isomeric SMILES for descriptor calculation
- name: Mordred V1.2.0
  role: Calculates the 1613 2D molecular descriptors for every queried candidate
- name: CCSP 2.0
  role: Consumes the candidate descriptor table to predict CCS values and filter candidates
    by ΔCCS threshold
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:c17caabdcca124900aa1358c5d9e9867dd394dd35b9506bbf8a75b13bf042d5d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - Biopython (Entrez)
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  - Mordred V1.2.0
  - PubChem
  - PubChemPy
  - Python
  - RDKit
  - SciKit-Learn
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
emails_redacted: true
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# pubchem-mass-based-candidate-querying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves all PubChem compound candidates matching a measured precursor m/z within a specified exact-mass tolerance (e.g. 10 ppm), then prepares them (isomeric SMILES, MOL files, Mordred descriptors) for CCS-based false-positive filtering as used in CCSP 2.0 metabolite annotation.

## When to use

Use when an unannotated IM-MS feature (e.g. [M-H]- at m/z 155.0093) has an experimentally measured CCS and you need all plausible PubChem structures within a 10 ppm exact-mass window to filter by predicted ΔCCS.

## When NOT to use

- The analyte is a dimer, less common adduct, or in-source fragment (e.g. [2M+H]+, [2M-H]-) — CCSP 2.0's training sets do not support these species
- No experimental CCS measurement exists for the feature, so the ΔCCS filtering rationale that motivates the candidate query cannot be applied
- Very large-scale screening through the Google Colab route — Colab limits continuous operation to ~12 hours and disconnects after ~20 minutes idle; use the local Jupyter Lab notebook instead

## Inputs

- Measured precursor m/z and adduct type (e.g. [M-H]-, m/z 155.0093)
- Experimentally measured CCS value (e.g. 120.22 Å2 for isoorotic acid) and chosen ΔCCS threshold (2–2.8%)
- Mass tolerance window (default 10 ppm; evaluate 10/7/5/3/1/0 ppm for ROC analysis)

## Outputs

- List of PubChem candidate CIDs with exact masses within the mass window (e.g. 221 candidates for m/z 155.0093)
- Isomeric SMILES strings / RDKit MOL files for each candidate
- Mordred descriptor table (1613 2D descriptors per candidate) ready for SVR CCS prediction
- Filtered candidate set within the ΔCCS threshold, plus filtering percentages per mass window

## How to apply

Query PubChem through the Entrez Global Query Cross-Database Search System (Biopython Entrez module) for all single compounds whose exact mass lies within the tolerance (10 ppm is the paper's default; windows down to 0 ppm are also evaluated). Expect large candidate sets — 43 to 13,984 candidates per compound (136,940 total for 33 lipids in the class-matched lipid validation). Retrieve each candidate CID's isomeric SMILES with PubChemPy (or convert neutral InChI strings via the PubChem Identifier Exchange Service), encode MOL files with RDKit, and compute 1613 2D molecular descriptors with Mordred V1.2.0 (dropping constant/inapplicable descriptors and Z-transforming the rest with training-set statistics). These descriptor tables are the direct input to the CCSP 2.0 linear-kernel SVR CCS predictor; candidates whose predicted CCS falls within the ΔCCS threshold (±2% in the isoorotic acid case; 2.8% for the lipid superclass model) of the measured CCS are retained as plausible annotations.

## Related tools

- **Biopython (Entrez)** (Issues the PubChem exact-mass candidate queries via the Entrez Global Query Cross-Database Search System)
- **PubChemPy** (Retrieves each candidate CID's isomeric SMILES from PubChem)
- **RDKit** (Encodes MOL files from the retrieved isomeric SMILES for descriptor calculation)
- **Mordred V1.2.0** (Calculates the 1613 2D molecular descriptors for every queried candidate)
- **CCSP 2.0** (Consumes the candidate descriptor table to predict CCS values and filter candidates by ΔCCS threshold) — https://github.com/facundof2016/CCSP2.0

## Examples

```
from Bio import Entrez; Entrez.email='<maintainer e-mail, see the repository README>'; cids = Entrez.read(Entrez.esearch(db='pccompound', term='155.0093[ExactMass]@10ppm', RetMax=10000))
```

## Evaluation signals

- Isoorotic acid benchmark: the 10 ppm query must return exactly 221 PubChem candidates, and exactly 32 are retained at a ±2% ΔCCS window (86% of false positives removed) with the correct annotation (predicted 119.3 vs experimental 120.22 Å2) kept
- Class-matched lipid benchmark: 100% of correct annotations retained at ΔCCS 2.8% and 10 ppm, with a median of 36.1% of candidates filtered (19.3% of isomeric candidates at 0 ppm)
- Lipid-class ROC AUC scores fall in the 0.748–0.776 range across mass windows
- Candidate counts per compound lie in the observed 43–13,984 range at 10 ppm; verify descriptors were Z-transformed with training-set statistics and constant/inapplicable descriptors were dropped
- Filtered set must still contain the true annotation — any run retaining 0 correct candidates indicates a threshold or descriptor-pipeline error

## Limitations

- The 10 ppm window can return over 13,000 candidates per compound, so large queries need the locally hosted Jupyter Lab notebook rather than Google Colab (12-hour limit, ~20-minute idle disconnect)
- Candidate retrieval and CCS filtering require Python familiarity, which the authors note limits accessibility
- Training sets support only [M-H]-, [M+H]+ and [M+Na]+ monomer adducts — not dimers, rare adducts, or in-source fragments
- The effect of experimental CCS variance on the threshold choice had not yet been investigated at the time of writing

## Evidence

- [results] isoorotic acid case parameters: [evidence span withheld — anchor status: not_found]
- [readme] CCSP 2.0 purpose: [evidence span withheld — anchor status: external_doc]
