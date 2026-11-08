---
name: pubchem-entrez-candidate-querying
description: Use when building an annotation-candidate list for a target analyte of
  known exact mass — e.g. gathering PubChem structures for downstream CCS prediction
  and ΔCCS filtering.
when_to_use_negative:
- If the target compounds already have curated candidate lists or if only the true-annotation
  CCS prediction is needed (no competitor-structure enumeration) — skip the broad
  Entrez query.
- When the target compound has no exact PubChem match; in the article 9 such compounds
  were excluded, so a missing record means the query step will return nothing useful.
- When the analysis requires dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source
  fragments — CCSP 2.0 training sets do not support these, so candidate enumeration
  for them is out of scope.
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_3335
tools:
- name: Biopython
  role: Provides the Entrez Global Query Cross-Database Search System interface used
    to execute the PubChem candidate queries at a 10 ppm exact-mass tolerance
- name: PubChem
  role: Target database queried to retrieve all candidate annotations for each target
    compound
- name: Mordred V1.2.0
  role: Used to compute the exact mass (and 1613 2D descriptors) from RDKit-encoded
    MOL files, defining the 10 ppm query window
- name: RDKit
  role: Encodes the retrieved candidate SMILES into MOL files that Mordred then processes
- name: CCSP 2.0
  role: 'Downstream consumer: predicted CCS values for each queried candidate enable
    ΔCCS filtering (e.g. 2.8–3% threshold)'
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:e1c7c1e61f7ffd17f47dccccf4cfd833452153cf14aa73abca9e42a85b60ec83
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
  - Biopython
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  - Mordred V1.2.0
  - PubChem
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_3335
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# pubchem-entrez-candidate-querying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Query PubChem via the Entrez Global Query Cross-Database Search System (Biopython) to enumerate all candidate annotations whose exact mass falls within a specified ppm tolerance of a target compound's exact mass, producing the candidate lists that feed downstream CCS-based filtering.

## When to use

Use when building an annotation-candidate list for a target analyte of known exact mass — e.g. gathering PubChem structures for downstream CCS prediction and ΔCCS filtering.

## When NOT to use

- If the target compounds already have curated candidate lists or if only the true-annotation CCS prediction is needed (no competitor-structure enumeration) — skip the broad Entrez query.
- When the target compound has no exact PubChem match; in the article 9 such compounds were excluded, so a missing record means the query step will return nothing useful.
- When the analysis requires dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments — CCSP 2.0 training sets do not support these, so candidate enumeration for them is out of scope.

## Inputs

- List of target compounds with known exact masses (e.g. 162 deprotonated [M-H]- standards from Supplemental Table 4)
- Mass-error tolerance in ppm (e.g. 10 ppm)
- Biopython Entrez access to the PubChem Global Query Cross-Database Search System
- Optional: the true compound's chemical formula for post-query candidate-list restriction

## Outputs

- Per-target PubChem candidate record sets (exact mass within tolerance), counts ranging 15–18,399 per compound
- Combined candidate set (e.g. 598,208 candidates across the 162-compound deprotonated test set)
- Per-candidate SMILES/MOL structures ready for RDKit encoding and Mordred descriptor calculation

## How to apply

For each target standard (e.g. the 162 deprotonated [M-H]- validation compounds from Supplemental Table 4), compute the exact mass (as calculated by Mordred from the RDKit-encoded structure) and issue a Biopython Entrez query against the PubChem Global Query Cross-Database Search System to retrieve every candidate annotation whose exact mass lies within the chosen mass-error window — 10 ppm in the CCSP 2.0 validation, matching the ΔCCS-threshold evaluation at 2.8–3%. Retrieve all matching candidate records (candidate counts ranged from 15 to 18,399 per compound, totaling 598,208 for the 162-compound set, after excluding the 9 compounds with no exact PubChem match). Then encode each candidate's isomeric SMILES into a MOL file with RDKit and calculate the 1613 2D descriptors via Mordred V1.2.0 so the candidates can be scored by the CCSP SVR model. Downstream, optionally restrict candidates to those sharing the true compound's chemical formula before filtering. The rationale is that candidate enumeration must be structurally unbiased at fixed mass tolerance so that CCS filtering percentages (e.g. 47.3% median filtered at ΔCCS 3%) are measured against a realistic search space.

## Related tools

- **Biopython** (Provides the Entrez Global Query Cross-Database Search System interface used to execute the PubChem candidate queries at a 10 ppm exact-mass tolerance)
- **PubChem** (Target database queried to retrieve all candidate annotations for each target compound)
- **Mordred V1.2.0** (Used to compute the exact mass (and 1613 2D descriptors) from RDKit-encoded MOL files, defining the 10 ppm query window)
- **RDKit** (Encodes the retrieved candidate SMILES into MOL files that Mordred then processes)
- **CCSP 2.0** (Downstream consumer: predicted CCS values for each queried candidate enable ΔCCS filtering (e.g. 2.8–3% threshold)) — https://github.com/facundof2016/CCSP2.0

## Evaluation signals

- Verify that every returned candidate's exact mass lies within the stated ppm tolerance of the target compound's Mordred-computed exact mass.
- Check candidate counts per compound fall in a plausible range (15–18,399 per compound in the reference run); flag any target with zero hits and exclude it as in the article's 9 compounds.
- Confirm the combined candidate total matches expectation for the given target set and tolerance (e.g. 598,208 for 162 deprotonated compounds at 10 ppm).
- Confirm each retrieved candidate has an isomeric SMILES that RDKit can convert to a MOL file and Mordred can process without error.
- Optionally, recompute candidate lists after restricting to candidates sharing the target's chemical formula and verify the resulting list size reduction (median 24.9% filtered among same-formula candidates at ΔCCS 3%).

## Limitations

- Compounds lacking an exact PubChem match cannot be handled — the workflow excluded 9 such validation compounds.
- Candidate list sizes vary enormously (15 to 18,399 per compound in the article), so query runtime and downstream descriptor calculation scale unpredictably with tolerance and target class.
- PubChem querying relies on exact-mass matching only at the query stage; structural diversity of the candidate pool can be very high, so subsequent CCS prediction and ΔCCS filtering (e.g. ΔCCS 3% with 76.5% true-annotation retention) is required to reduce false positives.

## Evidence
