---
name: spectral-library-curation
description: Use when building or auditing a merged MS/MS reference library from multiple
  source databases and you need to detect and quantify duplicate compounds before
  deduplication, using InChIKey identity across NIST20, Agilent METLIN, GNPS, MSDIAL.
when_to_use_negative:
- Input is already a deduplicated compound table or the source library attribution
  has already been applied — re-running the priority collapse would double-subtract
  shared compounds
- The goal is spectrum-level candidate identification (CNN embedding prediction, cosine-similarity
  matching) rather than library construction/audit — use the ChemEmbed prediction
  pipeline instead
- Compounds must be distinguished at the stereochemical layer (full 27-character InChIKey),
  since this skill deliberately compares only the 14-character connectivity prefix
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Load the Supplementary File 2.pkl deposit, truncate InChIKey strings to 14
    characters, and group/count compounds per source library
- name: RDKit
  role: Chemical-structure toolkit used in the same paper's pipeline (InChI/SMILES
    handling) supporting the compound-identity layer of the curation
- name: ChemEmbed
  role: 'Downstream consumer: the deduplicated compound set underpins the 520,083-compound
    Mol2vec-300d reference database used for cosine-similarity candidate matching'
  repo: https://github.com/faizanurv/ChemEmbed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:4a1676b9831497f0936d16bf79224fc9c90c3e4c66caecd78eee78d610c7ba55
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/massspecdl/ChemEmbed
  repo_url: https://github.com/faizanurv/ChemEmbed
  tools:
  - ChemEmbed
  - Python
  - Python+
  - RDKit
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectral-library-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies cross-library compound overlap (NIST20, Agilent METLIN, GNPS, MSDIAL) in an MS/MS spectral reference database by matching 14-character InChIKey connectivity-layer prefixes, then deduplicates under a fixed source-library priority hierarchy to produce a reconciled unique-compound count.

## When to use

Use when building or auditing a merged MS/MS reference library from multiple source databases and you need to detect and quantify duplicate compounds before deduplication, using InChIKey identity across NIST20, Agilent METLIN, GNPS, MSDIAL.

## When NOT to use

- Input is already a deduplicated compound table or the source library attribution has already been applied — re-running the priority collapse would double-subtract shared compounds
- The goal is spectrum-level candidate identification (CNN embedding prediction, cosine-similarity matching) rather than library construction/audit — use the ChemEmbed prediction pipeline instead
- Compounds must be distinguished at the stereochemical layer (full 27-character InChIKey), since this skill deliberately compares only the 14-character connectivity prefix

## Inputs

- Supplementary File 2.pkl from Zenodo deposit DOI 10.5281/zenodo.14778518 (compound records with InChIKey, SMILES, precursor masses and library-of-origin fields from NIST20, Agilent METLIN, GNPS, MSDIAL)
- ChemEmbed reference database parquet/pkl (chemembed_reference_mol2vec_300d_520k, 520,083 compounds) derived from the same Supplementary File 2.pkl

## Outputs

- overlap_counts.csv with per-library unique counts, pairwise overlap counts, and priority-attributed duplicate counts
- Deduplicated compound set reconciled against the reported 38,472 unique compounds for downstream reference-database construction

## How to apply

Load Supplementary File 2.pkl from the Zenodo deposit DOI 10.5281/zenodo.14778518 with pandas and extract the InChIKey field for every compound. Truncate each key to its 14-character prefix so comparisons operate at the connectivity layer (ignoring stereochemistry/protonation layers). Group compounds by source library and compute unique counts plus pairwise and multiway set intersections across the four libraries. Apply the priority hierarchy NIST20 > Agilent METLIN > GNPS > MSDIAL to attribute each shared compound to a single source library, keeping spectra only from the highest-priority library for each duplicated compound. Write per-library unique counts, pairwise overlap counts, and priority-attributed counts to overlap_counts.csv.

## Related tools

- **pandas** (Load the Supplementary File 2.pkl deposit, truncate InChIKey strings to 14 characters, and group/count compounds per source library)
- **RDKit** (Chemical-structure toolkit used in the same paper's pipeline (InChI/SMILES handling) supporting the compound-identity layer of the curation)
- **ChemEmbed** (Downstream consumer: the deduplicated compound set underpins the 520,083-compound Mol2vec-300d reference database used for cosine-similarity candidate matching) — https://github.com/faizanurv/ChemEmbed

## Examples

```
import pandas as pd; df = pd.read_pickle('Supplementary File 2.pkl'); df['ik14'] = df['InChIKey'].str[:14]; counts = df.groupby(['library'])['ik14'].nunique()
```

## Evaluation signals

- The deduplicated compound total across the four libraries must reconcile with the reported 38,472 unique compounds sourced from NIST20, MSDIAL, GNPS, and Agilent METLIN
- Per-library unique, pairwise-overlap, and priority-attributed counts must reproduce the values plotted in Supplementary Figure 1 and tabulated in Tables 3-4
- Every InChIKey string in the grouping key is exactly 14 characters after truncation (no nulls or full-length keys leaking into the key)
- No compound appears attributed to more than one source library after applying the NIST20 > Agilent METLIN > GNPS > MSDIAL priority
- overlap_counts.csv row counts equal the number of unique 14-character InChIKey prefixes observed across the four libraries

## Limitations

- Matching on the 14-character InChIKey prefix ignores stereochemistry and protonation layers, so stereoisomers are collapsed into one compound at the connectivity layer
- Candidate matching in the downstream reference can only return molecules present in the deduplicated database; compounds outside it cannot be identified regardless of spectral quality
- Results obtained with a differently sized reference database are not directly comparable, since the 520,083-compound reference is fixed to this curation

## Evidence
