---
name: inchikey-compound-deduplication
description: Use when merging MS/MS spectral reference libraries (NIST20, Agilent
  METLIN, GNPS, MSDIAL) and you need to quantify and remove cross-library duplicate
  compounds before building a combined database or reference file.
when_to_use_negative:
- When deduplicating a single library in isolation — the skill's priority hierarchy
  (NIST20 > Agilent METLIN > GNPS > MSDIAL) is only meaningful when merging multiple
  source libraries.
- When compounds must be distinguished at the full-InChIKey level (e.g. separating
  stereoisomers or protonation-layer duplicates); truncating to 14 characters merges
  entries that differ beyond the connectivity layer.
- When the goal is spectral similarity matching rather than compound identity bookkeeping
  — use cosine/FAISS candidate retrieval, not InChIKey prefix comparison.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3346
tools:
- name: ChemEmbed
  role: The deduplicated reference database is the compound reference used downstream
    by ChemEmbed's CNN-embedding candidate matching; the deduplication step is performed
    before building ChemEmbed's reference DB.
  repo: github.com/massspecdl/ChemEmbed
- name: pandas >= 1.0.0
  role: Loads the .pkl compound metadata, applies the 14-character InChIKey truncation
    and library grouping/intersection logic, and writes the overlap_counts.csv output.
- name: RDKit
  role: Used in the article's pipeline for identifier handling (InChI to SMILES conversion);
    usable for parsing/validating InChIKeys during deduplication.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.02.07.637102
    title: 'ChemEmbed: A deep learning framework for metabolite identification using
      enhanced MS/MS data and multidimensional molecular embeddings'
  source_content_hash: sha256:1fadf89eb40aa7cc9ba63dd9cc68fa473aa804802623f54a17b86281ec32b15f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/massspecdl/ChemEmbed
  tools:
  - ChemEmbed
  - Python+
  - RDKit
  - pandas >= 1.0.0
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3346
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# inchikey-compound-deduplication

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects compounds present in multiple source spectral libraries by matching InChIKey identifiers truncated to their 14-character connectivity-layer prefix, then attributes each duplicate to a single library under a fixed priority hierarchy (NIST20 > Agilent METLIN > GNPS > MSDIAL) to yield a deduplicated reference database.

## When to use

Use when merging MS/MS spectral reference libraries (NIST20, Agilent METLIN, GNPS, MSDIAL) and you need to quantify and remove cross-library duplicate compounds before building a combined database or reference file.

## When NOT to use

- When deduplicating a single library in isolation — the skill's priority hierarchy (NIST20 > Agilent METLIN > GNPS > MSDIAL) is only meaningful when merging multiple source libraries.
- When compounds must be distinguished at the full-InChIKey level (e.g. separating stereoisomers or protonation-layer duplicates); truncating to 14 characters merges entries that differ beyond the connectivity layer.
- When the goal is spectral similarity matching rather than compound identity bookkeeping — use cosine/FAISS candidate retrieval, not InChIKey prefix comparison.

## Inputs

- Supplementary File 2.pkl from Zenodo deposit DOI 10.5281/zenodo.14778518 (compound records with InChIKey, SMILES, precursor mass, and source library label)
- Optional equivalent: chemembed_reference_mol2vec_300d_520k.parquet (or .pkl), a 520,083-compound reference database with unchanged InChIKeys and library provenance

## Outputs

- overlap_counts.csv with per-library unique compound counts, pairwise overlap counts, and priority-attributed duplicate counts
- Deduplicated compound/spectrum table where each compound appears once, attributed to its highest-priority source library

## How to apply

Load the compound metadata (Supplementary File 2.pkl from Zenodo DOI 10.5281/zenodo.14778518, or the parquet conversion chemembed_reference_mol2vec_300d_520k.parquet) into pandas, and extract the InChIKey field for every compound. Truncate each InChIKey to its first 14 characters so compounds are compared at the connectivity layer rather than the protonation/stereochemistry layers. Group compounds by their source library (NIST20, Agilent METLIN, GNPS, MSDIAL) and compute unique per-library counts plus pairwise and multiway set intersections of the 14-character prefixes. Resolve duplicates by applying the priority order NIST20 > Agilent METLIN > GNPS > MSDIAL, keeping the spectrum/compound entry from the highest-priority library only, and write per-library unique counts, pairwise overlap counts, and priority-attributed counts to a tabular output (e.g. overlap_counts.csv). The deduplicated total must reconcile with the article's 38,472 unique compounds.

## Related tools

- **ChemEmbed** (The deduplicated reference database is the compound reference used downstream by ChemEmbed's CNN-embedding candidate matching; the deduplication step is performed before building ChemEmbed's reference DB.) — github.com/massspecdl/ChemEmbed
- **pandas >= 1.0.0** (Loads the .pkl compound metadata, applies the 14-character InChIKey truncation and library grouping/intersection logic, and writes the overlap_counts.csv output.)
- **RDKit** (Used in the article's pipeline for identifier handling (InChI to SMILES conversion); usable for parsing/validating InChIKeys during deduplication.)

## Examples

```
import pandas as pd; df = pd.read_pickle('Supplementary File 2.pkl'); df['inchikey14'] = df['InChIKey'].str[:14]; counts = df.groupby('library')['inchikey14'].nunique(); counts.to_csv('overlap_counts.csv')
```

## Evaluation signals

- The deduplicated compound total across the four libraries reconciles with the reported 38,472 unique compounds sourced from NIST20, MSDIAL, GNPS, and Agilent METLIN.
- Per-library unique counts, pairwise overlap counts, and priority-attributed counts reproduce the values plotted in Supplementary Figure 1 and tabulated in Tables 3-4.
- Every deduplicated compound appears exactly once and is attributed to the highest-priority library in the order NIST20 > Agilent METLIN > GNPS > MSDIAL.
- Each compound key in the output matches a 14-character prefix of a full InChIKey (structure-checkable with RDKit).
- The overlap_counts.csv contains columns for both pairwise and multiway intersections, not just per-library totals.

## Limitations

- Comparison at the 14-character connectivity layer collapses differences in the protonation/stereochemistry layers of the InChIKey, so compounds differing only beyond the first 14 characters are treated as duplicates.
- The priority hierarchy is a fixed editorial decision (NIST20 > Agilent METLIN > GNPS > MSDIAL); it does not guarantee the highest-priority library carries the best-quality spectrum for a given compound.
- Results are only reproducible against the specific reference files distributed on the v1.0.1 release or Zenodo DOI 10.5281/zenodo.14778518; differently sized reference sets are not directly comparable.

## Evidence
