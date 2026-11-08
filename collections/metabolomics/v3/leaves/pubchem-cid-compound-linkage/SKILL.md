---
name: pubchem-cid-compound-linkage
description: Use when building SQL-format compound database entries from reviewed
  CCSfind results and each molecular formula entry needs a PubChem CID cross-reference
  alongside its DTCCS N2 value, EICs and full IM spectra.
when_to_use_negative:
- Input is drift tube data outside the LC-DTIM-MS scope CCSfind targets, or no CCSfind-derived
  compound entries exist yet to annotate
- The database entries are already committed and only a bulk identifier export is
  needed rather than per-entry PubChem CID annotation
- The molecular formulae cannot be resolved to a PubChem compound (no valid CID),
  since the annotation step requires a formula-to-CID mapping
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: CCSfind
  role: Produces the reviewed compound entries that are annotated with PubChem CIDs
    and committed as SQL-format database entries
  repo: https://github.com/sangeeta97/ccs_find.git
- name: polars
  role: Loads the reviewed CCSfind results for assembly and annotation of compound
    records
- name: pandas
  role: Alternative DataFrame backend for handling the curated peak/feature data during
    entry assembly
- name: conda
  role: Creates the Python 3.9 environment in which CCSfind's processing and database-building
    steps run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:109ac928a0723ad6873d083a91f0892d40c6dce57de9cd31e55d5ea7ee05d4fa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sangeeta97/ccs_find
  repo_url: https://github.com/sangeeta97/ccs_find.git
  tools:
  - CCSfind
  - Python
  - conda
  - numpy
  - pandas
  - pip
  - polars
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# pubchem-cid-compound-linkage

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates each CCSfind compound entry — assembled from reviewed LC-DTIM-MS results with EICs, full IM spectra, DTCCS N2 values and experimental settings — with a PubChem CID link for its input molecular formula, so SQL-format database entries carry a chemical-identifier cross-reference.

## When to use

Use when building SQL-format compound database entries from reviewed CCSfind results and each molecular formula entry needs a PubChem CID cross-reference alongside its DTCCS N2 value, EICs and full IM spectra.

## When NOT to use

- Input is drift tube data outside the LC-DTIM-MS scope CCSfind targets, or no CCSfind-derived compound entries exist yet to annotate
- The database entries are already committed and only a bulk identifier export is needed rather than per-entry PubChem CID annotation
- The molecular formulae cannot be resolved to a PubChem compound (no valid CID), since the annotation step requires a formula-to-CID mapping

## Inputs

- Reviewed CCSfind results (curated peak/feature data from parsed profile mzML LC-DTIM-MS input)
- List of input molecular formulae with their PubChem CIDs
- Per-entry metadata: DTCCS N2 values, experimental settings from manual review, EICs and full IM spectra

## Outputs

- SQL-format database entries, each annotated with the PubChem CID for its molecular formula
- Separate database entries for chromatographically separated isomers and IM-separated protomers/conformers

## How to apply

After processing the parsed mzML input via 'python processor.py -g config.ini', load the reviewed CCSfind results with polars or pandas. For each compound record whose entry contains the extracted ion chromatograms (EICs), full normalized IM spectra, DTCCS N2 value and the experimental settings recorded during manual review, add the PubChem CID link for its input molecular formula. Encode chromatographically separated isomers and IM-separated protomers or conformers of the same ion species as separate entries, keeping user control over the SQL-format output. Commit the annotated records so the full dataset, including identifier metadata, can be retrieved from the database. Validate that each committed entry adds approximately 200 KB and that every entry resolves to exactly one PubChem CID per molecular formula.

## Related tools

- **CCSfind** (Produces the reviewed compound entries that are annotated with PubChem CIDs and committed as SQL-format database entries) — https://github.com/sangeeta97/ccs_find.git
- **polars** (Loads the reviewed CCSfind results for assembly and annotation of compound records)
- **pandas** (Alternative DataFrame backend for handling the curated peak/feature data during entry assembly)
- **conda** (Creates the Python 3.9 environment in which CCSfind's processing and database-building steps run)

## Examples

```
python processor.py -g config.ini
```

## Evaluation signals

- Every committed compound entry contains a valid PubChem CID linked to exactly one input molecular formula
- Each reviewed entry retains EICs, full normalized IM spectra, DTCCS N2 value and experimental settings, not only summary values
- Committing each reviewed compound entry increases the database file size by the reported moderate ~200 KB per entry
- Chromatographically separated isomers and IM-separated protomers/conformers appear as separate entries with distinct CIDs/formula-annotated records
- The full dataset is retrievable from the resulting SQL-format database

## Limitations

- Storage cost grows by approximately 200 KB per entry because full IM spectra are stored in the database, so large compound sets inflate database size
- The workflow depends on drift tube IM-MS (LC-DTIM-MS) profile mzML input with single-field CCS calibration coefficients (Tfix, Beta) and a known drift gas identity (e.g., nitrogen); other IM technologies are out of scope
- Encoding of isomers/protomers/conformers as separate entries is subject to user control over the output, so misconfiguration can merge or split entries incorrectly

## Evidence
