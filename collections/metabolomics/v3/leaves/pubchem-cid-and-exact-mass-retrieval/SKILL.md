---
name: pubchem-cid-and-exact-mass-retrieval
description: Use when processing an Agilent Unknowns Analysis GC-MS CSV (columns Component.RT,
  Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) and per-compound
  PubChem CIDs, exact masses, and names are needed to build the spread database.
when_to_use_negative:
- The data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms depend
  on spectra published in PubChem and do not yet apply to these acquisition modes
- The input is already a curated feature/annotation table with CIDs or exact masses
  — re-querying PubChem adds no value
- LC-MS or untested instrument data (Q-TOF, Orbitrap, TimsTOF HT, Astral) — uafR has
  not been validated in those contexts
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: uafR
  role: Provides spreadOut(), which expands the hit table and performs the per-compound
    annotation fetch and fallback resolution
  repo: github.com/castratton/uafR
- name: webchem
  role: Supplies get_cid() to convert compound names into PubChem CIDs during the
    spreadOut() annotation fetch
- name: PubChem
  role: Source of CID, exact monoisotopic mass, and preferred compound name for each
    tentative compound
- name: NCI CACTUS
  role: Fallback resolver that provides a canonical SMILES for compounds absent from
    PubChem
- name: R
  role: Execution environment in which the spread database list object is assembled
    and serialized
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:be1dc1a07df0ae859a68558ad98c4ca6a9f9cac28dc3081f5fbf1a259b3bb347
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NCI CACTUS
  - PubChem
  - R
  - uafR
  - webchem
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# PubChem CID and exact-mass retrieval

## Summary

Within uafR's spreadOut() step, tentative Compound.Name values from an Agilent Unknowns Analysis hit table are converted to PubChem CIDs (via webchem get_cid()) to fetch exact monoisotopic masses and preferred names, with an NCI CACTUS fallback for canonical SMILES when no CID exists.

## When to use

Use when processing an Agilent Unknowns Analysis GC-MS CSV (columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) and per-compound PubChem CIDs, exact masses, and names are needed to build the spread database.

## When NOT to use

- The data are MS2, MS3, or ion mobility + MS2 spectra — uafR's algorithms depend on spectra published in PubChem and do not yet apply to these acquisition modes
- The input is already a curated feature/annotation table with CIDs or exact masses — re-querying PubChem adds no value
- LC-MS or untested instrument data (Q-TOF, Orbitrap, TimsTOF HT, Astral) — uafR has not been validated in those contexts

## Inputs

- Agilent Unknowns Analysis library-hit CSV (e.g. gcms_hit_table.csv) with columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor
- Tentative compound names (Compound.Name values), optionally subset by a Match.Factor threshold

## Outputs

- Spread database (list-structured R object, e.g. spread_database.rds) with one record per datapoint and unique identifiers
- Per-compound PubChem CID, exact monoisotopic mass, and preferred compound name
- Canonical SMILES for compounds resolved via NCI CACTUS

## How to apply

Load the library-hit CSV and verify the six required columns are present, then run spreadOut() to expand every recorded peak into a uniquely identified record retaining component area, tentative identity, match factor, base-peak m/z, retention time, and sample ID. For each tentative compound name, query PubChem using webchem's get_cid() to retrieve the CID, exact monoisotopic mass, and preferred compound name; PubChem is the 'most advanced open-source chemical dataset' uafR relies on for m/z-spectra-based identification. For compounds absent from PubChem, fall back to the NCI CACTUS resolver to obtain a canonical SMILES, which is then used to simulate mass and structure data. Assemble all retrieved information as a list-structured spread database (e.g. serialized as spread_database.rds) so downstream mzExacto() and standardifyIt() steps can operate on it. Compounds that lack a strong cross-sample match, have no PubChem entry, or yield no canonical SMILES can subsequently be removed via the decontaminate step.

## Related tools

- **uafR** (Provides spreadOut(), which expands the hit table and performs the per-compound annotation fetch and fallback resolution) — github.com/castratton/uafR
- **webchem** (Supplies get_cid() to convert compound names into PubChem CIDs during the spreadOut() annotation fetch)
- **PubChem** (Source of CID, exact monoisotopic mass, and preferred compound name for each tentative compound)
- **NCI CACTUS** (Fallback resolver that provides a canonical SMILES for compounds absent from PubChem)
- **R** (Execution environment in which the spread database list object is assembled and serialized)

## Examples

```
input_dat = read.csv("gcms_hit_table.csv"); input_spread = spreadOut(input_dat)
```

## Evaluation signals

- Input CSV passes the schema check: all six required columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) present before spreadOut() runs
- No duplicate peak keys in the expanded table; every expanded record has a unique identifier
- Every resolved compound has a valid PubChem CID with matching exact monoisotopic mass and preferred name; unresolved compounds are flagged for the CACTUS fallback or decontaminate step
- Spread database serializes successfully as spread_database.rds and round-trips into mzExacto()
- Compound areas validated against chemical/biological sense — uafR explicitly requires manual validation of algorithm-identified areas

## Limitations

- Compounds not found in PubChem require the NCI CACTUS fallback; those lacking a CID, canonical SMILES, or strong cross-sample match may be removed by decontamination
- MS2/MS3 and ion mobility support is deferred until such spectra are published in PubChem's database
- Match-factor-based subsetting (e.g. Match.Factor > 80 or > 70) determines which compound names enter the retrieval, so low-match-factor compounds may never be annotated
- Researchers must still manually validate that identified compound areas make chemical and/or biological sense

## Evidence

- [methods] spreadOut() expands every recorded data point into a large database with unique IDs, and fetches per-compound annotations by converting chemical names into PubChem CIDs using webchem's get_cid(): [evidence span withheld — anchor status: not_found]
- [intro] communicates with public chemistry utilities (intro): "communicates with public chemistry utilities (including PubChem and the National Cancer Institute) to sort and process the aggregated set of all tentatively identified molecules"
- [readme] required column names (readme): [evidence span withheld — anchor status: external_doc]
- [results] PubChem as the open-source dataset (results): "This package harnesses the most advanced open-source chemical dataset and makes it accessible to anyone with basic experience working in R."
