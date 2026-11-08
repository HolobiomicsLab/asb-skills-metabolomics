---
name: jaccard-similarity-scoring-of-molecular-structures
description: Use when benchmarking spectral similarity or alignment algorithms and
  a per-pair structural similarity ground truth is needed for compound pairs with
  known SMILES from GNPS library metadata or MassBank records.
when_to_use_negative:
- The analysis goal is spectral similarity scoring itself (e.g., SIMILE z_test, modified
  cosine) rather than constructing the structural ground-truth benchmark
- Compound structures lack parseable SMILES, since RDKit molecule objects cannot be
  built and the MCS search cannot run
- A ring-aware substructure definition is required, since this skill fixes ringMatchesRingOnly=False
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: RDKit
  role: Parses SMILES into molecule objects and performs the per-pair maximum common
    substructure search (ringMatchesRingOnly=False, timeout=1800 s) used to derive
    Jaccard coefficients
- name: pandas
  role: Loads the compound-structure pairs from file and writes the per-pair MCS Jaccard
    coefficients to mcs_results.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:4c3fa0b9084327488870fd8768d9634de99635100cdec76d0c755b4b6ae9d1db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RDKit
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# jaccard-similarity-scoring-of-molecular-structures

## Summary

Computes an all-vs-all maximum common substructure (MCS) Jaccard similarity matrix from compound SMILES using RDKit, producing the structural ground truth against which spectral alignment algorithms (e.g., SIMILE, modified cosine) are benchmarked.

## When to use

Use when benchmarking spectral similarity or alignment algorithms and a per-pair structural similarity ground truth is needed for compound pairs with known SMILES from GNPS library metadata or MassBank records.

## When NOT to use

- The analysis goal is spectral similarity scoring itself (e.g., SIMILE z_test, modified cosine) rather than constructing the structural ground-truth benchmark
- Compound structures lack parseable SMILES, since RDKit molecule objects cannot be built and the MCS search cannot run
- A ring-aware substructure definition is required, since this skill fixes ringMatchesRingOnly=False

## Inputs

- Compound-structure table of SMILES strings sourced from GNPS library metadata or MassBank records
- Spectra/compound pairs grouped by ionization mode (processed independently per mode)

## Outputs

- mcs_results.csv containing a per-pair MCS Jaccard coefficient (or timeout flag) for every input pair

## How to apply

Load the compound-structure table (SMILES strings sourced from GNPS library metadata or MassBank records) with pandas, parse each SMILES into an RDKit molecule object, and run an all-vs-all MCS calculation across all pairs of spectra/compounds separately for each ionization mode. For each pair, run RDKit's MCS search configured with ringMatchesRingOnly=False and a per-pair timeout of 1800 seconds. Convert the MCS result to the Jaccard coefficient s = N/((NA+NB)-N), where N is the atom count of the common substructure and NA, NB are the atom counts of the two structures. Write the per-pair coefficients (or timeout flags) to mcs_results.csv. This structural matrix underpins the SIMILE paper's claim that significant alignment scores correspond to compound structural similarity.

## Related tools

- **RDKit** (Parses SMILES into molecule objects and performs the per-pair maximum common substructure search (ringMatchesRingOnly=False, timeout=1800 s) used to derive Jaccard coefficients)
- **pandas** (Loads the compound-structure pairs from file and writes the per-pair MCS Jaccard coefficients to mcs_results.csv)

## Examples

```
from rdkit.Chem import rdFMCS; mcs = rdFMCS.FindMCS([m1, m2], ringMatchesRingOnly=False, timeout=1800)
```

## Evaluation signals

- Output table contains an MCS entry (coefficient or timeout flag) for every input pair — no missing pairs
- All computed Jaccard coefficients fall within [0, 1]
- Pairs with identical structures yield s = 1
- Symmetry check: s(i,j) equals s(j,i) for every compound pair in the all-vs-all calculation
- Pairs exceeding the 1800-second MCS search timeout are flagged rather than silently dropped

## Limitations

- Per-pair timeout of 1800 seconds means some structurally complex pairs are returned as timeout flags instead of coefficients
- The MCS ground truth is computed separately per ionization mode and does not itself capture spectral behavior
- Significant SIMILE alignment of fragmentation spectra does not necessitate compound structure similarity, so this ground truth supports correlation-based benchmarking, not one-to-one equivalence

## Evidence

- [methods] For each ionization mode, an all-vs-all calculation across all pairs of spectra/compounds was performed to compute the maximum common substructure (MCS) Jaccard similarity coefficient for every pair.: [evidence span withheld — anchor status: not_found]
- [methods] MCS search configuration: [evidence span withheld — anchor status: not_found]
- [methods] Validation of output table: [evidence span withheld — anchor status: not_found]
- [discussion] Structural similarity corresponds to alignment significance: [evidence span withheld — anchor status: not_found]
