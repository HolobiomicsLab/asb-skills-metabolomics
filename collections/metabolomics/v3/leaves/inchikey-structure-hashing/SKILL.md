---
name: inchikey-structure-hashing
description: Use when a spectral or metabolite library (e.g., GNPS .mgf entries with
  structural identifiers) needs unique structural keys so duplicate structures and
  stereoisomers can be collapsed before benchmarking or annotation.
when_to_use_negative:
- When stereoisomers must be kept distinct — the first-layer (14-character) deduplication
  intentionally merges stereoisomers
- When entries lack structural identifiers (e.g., adduct entries without structures)
  — these are excluded upstream before hashing
- When the goal is full-structure identity including stereochemistry — use the full
  27-character InChIKey rather than the first layer
tools:
- name: RDKit
  role: Converts each retained structure to its InChIKey for deduplication
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:d1740519ada68808a8363486c9d3c3895e11bdaf920834447363a281371472d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# inchikey-structure-hashing

## Summary

Converts chemical structures from library entries (e.g., GNPS spectral library metadata) to their 27-character InChIKey using RDKit, enabling structure deduplication and stereoisomer collapsing via the first InChIKey layer.

## When to use

Use when a spectral or metabolite library (e.g., GNPS .mgf entries with structural identifiers) needs unique structural keys so duplicate structures and stereoisomers can be collapsed before benchmarking or annotation.

## When NOT to use

- When stereoisomers must be kept distinct — the first-layer (14-character) deduplication intentionally merges stereoisomers
- When entries lack structural identifiers (e.g., adduct entries without structures) — these are excluded upstream before hashing
- When the goal is full-structure identity including stereochemistry — use the full 27-character InChIKey rather than the first layer

## Inputs

- Cleaned GNPS library entries with structural identifiers (e.g., SMILES or equivalent structure fields from parsed .mgf metadata)
- Structural identifiers for candidate metabolites requiring unique keys

## Outputs

- 27-character InChIKey per structure
- Deduplicated structure table keyed on the first 14 InChIKey characters (first layer), e.g., gnps_library_filtered.tsv with 40,138 entries / 8,558 unique first-layer InChIKeys

## How to apply

Parse each cleaned library entry's structural identifier and convert it to its InChIKey, a 27-character hashed version of the full InChI, using RDKit's InChI functionality. Then deduplicate by keeping distinct InChIKey values according to the first layer (first 14 characters), which deliberately removes duplicate structures AND stereoisomers. This step follows metadata parsing/cleaning (e.g., retention of [M+H]+ protonated adducts with structural identifiers) and precedes parent-mass filtering (100–1,500 Da) in the GNPS preprocessing workflow. In the source paper this produced 40,138 entries covering 8,558 unique structures from 66,646 concatenated GNPS library entries, yielding the file gnps_library_filtered.tsv.

## Related tools

- **RDKit** (Converts each retained structure to its InChIKey for deduplication)

## Examples

```
from rdkit import Chem; from rdkit.Chem.inchi import MolToInchiKey; ik = MolToInchiKey(Chem.MolFromSmiles(smiles)); key = ik[:14]
```

## Evaluation signals

- Every retained entry has a syntactically valid 27-character InChIKey
- Deduplicated table contains no repeated first-layer (first 14 characters) InChIKey values
- Applied to the GNPS workflow, the output gnps_library_filtered.tsv must contain 40,138 spectral entries over 8,558 unique first-layer InChIKeys
- All deduplicated entries satisfy downstream constraints: parent masses within 100–1,500 Da and spectra meeting the ≥6-fragment / 500-peak criteria
- Entries lacking structural identifiers or non-[M+H]+ adducts are absent from the hashed output

## Limitations

- First-layer (14-character) deduplication merges stereoisomers, so stereochemical distinction is lost in the final dataset
- The resulting benchmarking set is biased: all compounds are present in the DNP, favoring ISDB-DNP, and GNPS spectral libraries were also part of the Sirius training set
- 197 benchmarking entries had large mass differences (> 0.01 Da) between experimental parent ion mass and calculated exact mass; problematic entries are listed online (Problematic_entries.csv)

## Evidence

- [discussion] rdkit_inchikey_conversion: "converting structures to their InChIKey, a 27-character hashed version of the full InChI"
- [discussion] inchikey_dedup_first_layer: "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [other] gnps_dataset_validation_counts: [evidence span withheld — anchor status: not_found]
