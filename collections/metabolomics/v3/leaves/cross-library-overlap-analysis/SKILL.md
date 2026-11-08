---
name: cross-library-overlap-analysis
description: Reference for aggregating multiple MS/MS spectral libraries (e.g., NIST20,
  METLIN, GNPS, MSDIAL) into a deduplicated reference set, and for quantifying pairwise
  or multiway overlap via InChIKey prefix matching before assigning each compound
  to a single source library.
when_to_use_negative:
- Input is a single spectral library — overlap analysis requires at least two sources
  to be meaningful.
- Compounds lack InChIKey identifiers or use incomplete InChIKeys shorter than 14
  characters, preventing connectivity-layer matching.
- You need to resolve duplicates by spectral similarity rather than a fixed library
  priority order.
edam_operation: http://edamontology.org/operation_3453
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: ChemEmbed
  role: Reference implementation and paper source; its reference database (Supplementary
    File 2.pkl) is the input compound set being deduplicated
  repo: https://github.com/faizanurv/ChemEmbed
- name: pandas >= 1.0.0
  role: Loads the .pkl reference database and computes group-wise counts and set intersections
- name: RDKit
  role: Used in the broader ChemEmbed pipeline to convert InChI identifiers to SMILES;
    useful if source records need structural normalization before InChIKey extraction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:b4550c594bb8dd683a23227342e768880f2865893b5f66e6ebd2d8a9f087c001
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
  - Python+
  - RDKit
  - pandas >= 1.0.0
  edam_operation: http://edamontology.org/operation_3453
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cross-library-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

This skill quantifies compound overlap across multiple MS/MS spectral reference libraries (NIST20, Agilent METLIN, GNPS, MSDIAL) by matching the first 14 characters of InChIKey identifiers, then applies a deterministic priority hierarchy (NIST20 > Agilent METLIN > GNPS > MSDIAL) to resolve duplicates and produce a deduplicated compound database. It exists because naive merging of libraries inflates compound counts through cross-source duplication.

## When to use

Reference for aggregating multiple MS/MS spectral libraries (e.g., NIST20, METLIN, GNPS, MSDIAL) into a deduplicated reference set, and for quantifying pairwise or multiway overlap via InChIKey prefix matching before assigning each compound to a single source library.

## When NOT to use

- Input is a single spectral library — overlap analysis requires at least two sources to be meaningful.
- Compounds lack InChIKey identifiers or use incomplete InChIKeys shorter than 14 characters, preventing connectivity-layer matching.
- You need to resolve duplicates by spectral similarity rather than a fixed library priority order.

## Inputs

- Supplementary File 2.pkl (combined MS/MS spectral database with InChIKey, SMILES, precursor mass, and source-library fields)
- Library-source label per compound (NIST20, Agilent METLIN, GNPS, MSDIAL)

## Outputs

- overlap_counts.csv with per-library unique counts, pairwise overlap counts, and priority-attributed deduplicated counts
- Deduplicated compound list (38,472 unique compounds) suitable as a reference database for downstream spectral matching

## How to apply

Load the combined reference database (Supplementary File 2.pkl from the Zenodo deposit, DOI 10.5281/zenodo.14778518) with pandas. Truncate each compound's InChIKey to its first 14 characters — the connectivity layer — so structurally identical compounds from different libraries are compared independent of stereochemistry or salt/protonation differences. Group compounds by source library and compute unique counts plus pairwise/multiway set intersections across NIST20, Agilent METLIN, GNPS, and MSDIAL. Apply the priority hierarchy NIST20 > Agilent METLIN > GNPS > MSDIAL to attribute each shared compound to exactly one library, keeping spectra only from the highest-priority source. Write per-library unique counts, pairwise overlap counts, and priority-attributed counts to a CSV (e.g., overlap_counts.csv). Validate by reconciling the deduplicated total against the reported 38,472 unique compounds and by reproducing the values shown in Supplementary Figure 1 and Tables 3–4.

## Related tools

- **ChemEmbed** (Reference implementation and paper source; its reference database (Supplementary File 2.pkl) is the input compound set being deduplicated) — https://github.com/faizanurv/ChemEmbed
- **pandas >= 1.0.0** (Loads the .pkl reference database and computes group-wise counts and set intersections)
- **RDKit** (Used in the broader ChemEmbed pipeline to convert InChI identifiers to SMILES; useful if source records need structural normalization before InChIKey extraction)

## Examples

```
import pandas as pd
# Load the combined reference database
db = pd.read_pickle('data_model_files/chemembed_reference_mol2vec_300d_520k.pkl')
# Truncate InChIKey to connectivity layer
db['inchikey14'] = db['inchikey'].str[:14]
# Group by source library and count unique compounds
overlap = db.groupby('source_library')['inchikey14'].nunique()
overlap.to_csv('overlap_counts.csv')
```

## Evaluation signals

- The deduplicated compound total across the four libraries must equal the reported 38,472 unique compounds.
- Per-library unique counts must reproduce the values plotted in Supplementary Figure 1 and tabulated in Tables 3–4.
- Every duplicate InChIKey-14 cluster should be attributed to exactly one source library under the NIST20 > Agilent METLIN > GNPS > MSDIAL hierarchy.
- Sum of per-library priority-attributed counts must equal the total deduplicated compound count (no double-counting).

## Limitations

- Priority-based deduplication discards spectra from lower-priority libraries entirely for duplicated compounds — it does not select the highest-quality spectrum among duplicates.
- Matching relies on the 14-character InChIKey connectivity layer; stereoisomers and protonation-state variants collapse together, which may over-merge distinct spectra.
- Results depend on the exact library versions present in Supplementary File 2.pkl; updating any source library will change overlap counts and the deduplicated total.
- The paper does not document the deduplication logic in the README — it must be reimplemented from the workflow description.

## Evidence

- [methods] Cross-library duplicate resolution: [evidence span withheld — anchor status: not_found]
