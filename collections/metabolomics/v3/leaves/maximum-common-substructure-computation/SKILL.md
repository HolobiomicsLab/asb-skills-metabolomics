---
name: maximum-common-substructure-computation
description: Use when a study needs compound-level structural similarity ground truth
  to benchmark spectral alignment or spectral similarity scores (e.g., modified cosine,
  SIMILE p-values) against chemical structure.
when_to_use_negative:
- When the input is mass spectra rather than structures — use spectral similarity/alignment
  scores (cosine, modified cosine, SIMILE) instead, since MCS requires parsed molecular
  structures
- When a faster fingerprint-based structural similarity (e.g., Tanimoto on Morgan
  fingerprints) is sufficient, since exhaustive MCS with a 1800 s per-pair timeout
  is computationally expensive
- When pairs contain stereochemical or salt-form nuances that RDKit's default MCS
  matching with ringMatchesRingOnly=False would mis-handle
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3314
tools:
- name: RDKit
  role: Parses SMILES into molecule objects and performs the MCS search (ringMatchesRingOnly=False,
    1800 s per-pair timeout) from which Jaccard coefficients are derived
- name: pandas
  role: Loads the compound-SMILES table and writes the per-pair MCS results to mcs_results.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:c6abcbe854ac94cf4a6b51cab1792080c9e214693c8e3ff89cd509d3cfbeabfc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RDKit
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# maximum-common-substructure-computation

## Summary

Computes an all-vs-all maximum common substructure (MCS) Jaccard similarity matrix between compounds using RDKit, serving as ground-truth structural similarity when benchmarking spectral alignment algorithms such as SIMILE against cosine-based scores.

## When to use

Use when a study needs compound-level structural similarity ground truth to benchmark spectral alignment or spectral similarity scores (e.g., modified cosine, SIMILE p-values) against chemical structure.

## When NOT to use

- When the input is mass spectra rather than structures — use spectral similarity/alignment scores (cosine, modified cosine, SIMILE) instead, since MCS requires parsed molecular structures
- When a faster fingerprint-based structural similarity (e.g., Tanimoto on Morgan fingerprints) is sufficient, since exhaustive MCS with a 1800 s per-pair timeout is computationally expensive
- When pairs contain stereochemical or salt-form nuances that RDKit's default MCS matching with ringMatchesRingOnly=False would mis-handle

## Inputs

- Compound-structure table of SMILES strings sourced from GNPS library metadata or MassBank records (loaded via pandas)
- Ionization-mode grouping information for the spectra/compounds

## Outputs

- mcs_results.csv containing a per-pair MCS Jaccard coefficient (or timeout flag) for every input pair
- All-vs-all MCS Jaccard similarity matrix serving as ground-truth structural similarity

## How to apply

Load compound-structure pairs (SMILES from GNPS library metadata or MassBank records) with pandas and parse each SMILES into an RDKit molecule object. For each ionization mode, run an all-vs-all pairwise MCS search with RDKit's FindMCS configured with ringMatchesRingOnly=False and a per-pair timeout of 1800 seconds to bound pathological searches. Convert each MCS result to a Jaccard coefficient s = N/((NA+NB)−N), where N is the atom count of the maximum common substructure and NA, NB are the atom counts of the two structures. Write all per-pair coefficients to mcs_results.csv, retaining a timeout flag for pairs that exceeded the search limit. This matrix then acts as the structural ground truth against which significant SIMILE alignments or cosine similarities are compared, since the SIMILE paper evaluates whether significant alignment scores correspond to compound structural similarity.

## Related tools

- **RDKit** (Parses SMILES into molecule objects and performs the MCS search (ringMatchesRingOnly=False, 1800 s per-pair timeout) from which Jaccard coefficients are derived)
- **pandas** (Loads the compound-SMILES table and writes the per-pair MCS results to mcs_results.csv)

## Examples

```
from rdkit.Chem import rdFMCS; mcs = rdFMCS.FindMCS([molA, molB], ringMatchesRingOnly=False, timeout=1800); s = mcs.numAtoms / (molA.GetNumAtoms() + molB.GetNumAtoms() - mcs.numAtoms)
```

## Evaluation signals

- Verify mcs_results.csv contains an MCS entry (coefficient or timeout flag) for every input pair — no missing pairs in the all-vs-all table
- All computed Jaccard coefficients must fall within [0, 1]
- Pairs with identical structures must yield s = 1 exactly
- Timeout-flagged pairs should be sparse; a large fraction of 1800 s timeouts indicates the molecule set or search configuration needs revisiting
- Downstream sanity check: significant SIMILE alignment scores should correspond to moderate-to-high MCS Jaccard values, matching the paper's finding that significant alignments often reflect moderate-to-high structural similarity

## Limitations

- The 1800 s per-pair timeout means some structurally distant pairs may be left unresolved (flagged) rather than assigned a true Jaccard coefficient
- Significant spectral alignment (e.g., SIMILE p-values) does not necessitate compound structure similarity, so the MCS matrix is a ground truth for correlation analysis, not a one-to-one predictor of alignment significance
- SMILES quality depends on upstream metadata (GNPS library or MassBank records); parsing failures or inconsistent representation bias the all-vs-all coefficients

## Evidence

- [methods] For each ionization mode, an all-vs-all calculation across all pairs of spectra/compounds was performed to compute the maximum common substructure (MCS) Jaccard similarity coefficient for every pair.: [evidence span withheld — anchor status: not_found]
- [discussion] Significant SIMILE alignments often correspond to moderate-to-high structural similarity: "Nevertheless, the results presented here show that significant fragmentation-spectra alignments often correspond to moderate-to-high structural similarity."
- [discussion] Significant SIMILE alignment does not necessitate compound structure similarity: "It is important to note that significant SIMILE alignment of fragmentation spectra does not necessitate compound structure similarity."
- [discussion] Significant SIMILE alignment scores correspond to compound structural similarity: [evidence span withheld — anchor status: not_found]
