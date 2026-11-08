---
name: splash-key-deduplication
description: Use when integrating or repeatedly re-processing spectra from heterogeneous
  mass spectral libraries (.MSP, .MGF, .JSON, .CSV) and you need database-independent
  spectral identifiers plus skip-if-seen incremental updates.
when_to_use_negative:
- When the task is to deduplicate compounds by chemical structure only (identical
  InChIKeys) rather than spectra — FragHub handles structure dedup as a separate downstream
  step in the partitioning stage.
- When spectra lack peak lists or have been so heavily pre-filtered that the peak
  array is empty — SPLASH keys are computed from m/z:intensity pairs and would be
  meaningless or colliding.
- 'When you need cross-run identity of collision-energy-aware spectra: FragHub explicitly
  does not harmonize collision energies (around 70 different formats), so SPLASH keys
  may not discriminate spectra differing only in collision energy encoding.'
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FragHub
  role: Generates a SPLASH key for each parsed spectrum, stores all keys in 'update.json',
    and skips already-seen keys so only new spectra are processed on library updates
  repo: github.com/eMetaboHUB/FragHub
- name: spectra-hash (SPLASH)
  role: Reference implementations (Java, Python, C++, C#, R) of the SPLASH spectral
    hash algorithm used as the identifier scheme; provides a REST validation service
  repo: github.com/berlinguyinca/spectra-hash
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c02219
    title: 'FragHub: A Mass Spectral Library Data Integration Workflow'
  source_content_hash: sha256:35a1745e9f8f710f7f5d89fa3c7b3ff230a13c87f39a23e5556135d4fec2e6a7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/eMetaboHUB/FragHub
  tools:
  - Flash Entropy Search
  - FragHub
  - MS-Dial
  - Python
  - spectra-hash (SPLASH)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# splash-key-deduplication

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a SPLASH (SPectraL hASH) identifier for every mass spectrum in an OMSL integration workflow (FragHub) and persists all keys in an 'update.json' repository so that, on re-runs or library updates, only spectra whose SPLASH key is absent are processed — enabling incremental, database-independent spectral deduplication.

## When to use

Use when integrating or repeatedly re-processing spectra from heterogeneous mass spectral libraries (.MSP, .MGF, .JSON, .CSV) and you need database-independent spectral identifiers plus skip-if-seen incremental updates.

## When NOT to use

- When the task is to deduplicate compounds by chemical structure only (identical InChIKeys) rather than spectra — FragHub handles structure dedup as a separate downstream step in the partitioning stage.
- When spectra lack peak lists or have been so heavily pre-filtered that the peak array is empty — SPLASH keys are computed from m/z:intensity pairs and would be meaningless or colliding.
- When you need cross-run identity of collision-energy-aware spectra: FragHub explicitly does not harmonize collision energies (around 70 different formats), so SPLASH keys may not discriminate spectra differing only in collision energy encoding.

## Inputs

- Parsed mass spectra from OMSL files (.MSP, .MGF, .JSON, .CSV) with peak lists (m/z and intensity pairs)
- Existing 'update.json' SPLASH-key repository from a prior FragHub run (optional; created empty for a new project)
- In-house library files with '_insilico' or '_GC' filename suffixes when acquisition type or chromatographic mode is not encoded per-spectrum

## Outputs

- 'update.json' file containing the SPLASH key of every spectrum processed to date
- Set of only-the-new spectra forwarded to metadata cleaning, filtering, and partitioned output library files (.MSP, .MGF, .JSON, .CSV)

## How to apply

For each parsed spectrum, compute the SPLASH key from its peak list (m/z:intensity pairs) following the Wohlgemuth et al. algorithm (e.g. via the spectra-hash reference implementations, format 'splash10-0002-0900000000-b112e4e059e1ecf98c5f' with identifier, two summary blocks and a unique hash block). Record every generated key in the project's 'update.json' file, which FragHub's sixth GUI tab maintains as the repository of already-treated spectra. On a subsequent run against the same output directory, look up each incoming spectrum's SPLASH key in 'update.json' and skip it if already present, so only new spectra are processed and appended to previous FragHub outputs. Deduplicated spectra are then subject to the same downstream metadata cleaning (RDKit SMILES/InChI/InChIKey verification) and partitioning by ionization mode, GC/LC and experimental-vs-predicted origin. If you want to start the project over, use the reset option to delete stored splash keys and output files rather than manually editing 'update.json'.

## Related tools

- **FragHub** (Generates a SPLASH key for each parsed spectrum, stores all keys in 'update.json', and skips already-seen keys so only new spectra are processed on library updates) — github.com/eMetaboHUB/FragHub
- **spectra-hash (SPLASH)** (Reference implementations (Java, Python, C++, C#, R) of the SPLASH spectral hash algorithm used as the identifier scheme; provides a REST validation service) — github.com/berlinguyinca/spectra-hash

## Evaluation signals

- After a run, every processed spectrum has a SPLASH key recorded in 'update.json'; re-running FragHub on the same inputs against the same output directory processes zero new spectra.
- A generated SPLASH key conforms to the four-block format, e.g. 'splash10-0002-0900000000-b112e4e059e1ecf98c5f' (identifier block, two summary blocks, unique hash block).
- The integrated library reflects the article's dedup effect: input of 794,985 spectra / 35,673 unique chemical identifiers is refined to 602,744 spectra / 32,193 unique chemicals.
- Deleting a spectrum's key from 'update.json' (or resetting the project) causes that spectrum to be reprocessed on the next run, confirming the skip logic reads the repository.
- Identical spectra from different source OMSLs (e.g. GNPS, MoNA, MassBank, MS-Dial-VS17) collapse to a single SPLASH key rather than being integrated twice.

## Limitations

- The dedup repository is tied to the selected output project directory: 'DO NOT DELETE FILES INTO YOUR SELECTED OUTPUT DIRECTORY AFTER COPY CLEANED VERSIONS.' — deleting output files breaks incremental skip behavior unless you reset the project.
- Starting a new project requires the reset option to delete stored splash keys and output files; otherwise stale keys suppress processing of legitimately re-integrated spectra.
- Deduplication at the spectrum level via SPLASH does not resolve chemical-level redundancy; the workflow additionally removes spectra with identical InChIKeys and fragment lists during output partitioning.
- Collision energy harmonization was not addressed due to ~70 different nonstandardized measures, so SPLASH-based dedup cannot distinguish spectra whose collision-energy annotations differ in encoding.

## Evidence

- [readme] SPLASH as a database-independent spectral identifier analogous to InChIKey: [evidence span withheld — anchor status: external_doc]
