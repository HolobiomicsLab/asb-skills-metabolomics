---
name: peak-record-unique-identifier-assignment
description: Use when a raw Agilent Unknowns Analysis CSV (columns Component.RT, Component.Area,
  Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) must be expanded into one
  uniquely-identified record per datapoint before compound extraction or annotation.
when_to_use_negative:
- The input is already a spread database with unique per-datapoint identifiers (re-running
  would duplicate the expansion step)
- Data comes from LC-MS, Q-TOF, Orbitrap, TimsTOF HT, Astral, or MS2/MS3/ion mobility
  acquisitions — uafR has not been tested in those contexts and its algorithms do
  not yet apply to MS2/MS3 data
- The goal is only compound annotation without peak-table expansion (use categorate()/pubchemProfile()
  directly on compound names instead)
tools:
- name: uafR
  role: Provides spreadOut(), the function that expands the Unknowns Analysis hit
    table into uniquely-identified per-datapoint records and stores the list-structured
    spread database
  repo: github.com/castratton/uafR
- name: R
  role: Runtime environment in which the spreadOut() expansion and annotation-fetch
    step is executed
- name: webchem
  role: Supplies get_cid() to convert each tentative Compound.Name into a PubChem
    CID, from which exact monoisotopic mass and preferred name are fetched
- name: NCI CACTUS
  role: Fallback chemical resolver generating a canonical SMILES for compounds absent
    from PubChem, used to simulate mass and structure data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0306202
    title: 'uafR: An R package that automates mass spectrometry data processing'
  source_content_hash: sha256:a38c2c44100962006d92f297fef9ab531aff066e60884a6ea5305168e38e250a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NCI CACTUS
  - R
  - uafR
  - webchem
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# peak-record-unique-identifier-assignment

## Summary

Expands an Agilent Unknowns Analysis GC-MS library-hit table into a uniquely-identified record per datapoint (uafR's spreadOut() step), enriching every tentative compound with PubChem CID/monoisotopic mass or an NCI CACTUS canonical SMILES fallback, and stores the result as a list-structured spread database.

## When to use

Use when a raw Agilent Unknowns Analysis CSV (columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) must be expanded into one uniquely-identified record per datapoint before compound extraction or annotation.

## When NOT to use

- The input is already a spread database with unique per-datapoint identifiers (re-running would duplicate the expansion step)
- Data comes from LC-MS, Q-TOF, Orbitrap, TimsTOF HT, Astral, or MS2/MS3/ion mobility acquisitions — uafR has not been tested in those contexts and its algorithms do not yet apply to MS2/MS3 data
- The goal is only compound annotation without peak-table expansion (use categorate()/pubchemProfile() directly on compound names instead)

## Inputs

- Agilent Unknowns Analysis library-hit CSV (e.g. gcms_hit_table.csv) with columns Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor
- Optional user-specified output object for the list-structured spread database (serialized as spread_database.rds)

## Outputs

- Expanded spread database: one record per datapoint with unique identifiers, retaining component area, tentative compound name, match factor, base-peak m/z, retention time, and sample ID
- Per-compound PubChem annotations: CID, exact monoisotopic mass, preferred compound name (via webchem get_cid())
- Canonical SMILES from NCI CACTUS for compounds missing a PubChem CID
- Serialized list-structured spread database (spread_database.rds) consumable by mzExacto()

## How to apply

Load the Agilent Unknowns Analysis library-hit CSV (e.g. gcms_hit_table.csv) and verify the strict column requirements: Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, and Match.Factor must be present in any order. Reject duplicate peak keys, then expand the table into one record per datapoint, each carrying a unique identifier while retaining component area, tentative identity, match factor, base-peak m/z, retention time, and sample ID (File.Name). For every tentative Compound.Name, query PubChem via webchem::get_cid() to retrieve the CID, exact monoisotopic mass, and preferred compound name; the rationale is to attach published physicochemical metadata to each peak record before downstream matching. For compounds absent from PubChem, fall back to the NCI CACTUS resolver to generate a canonical SMILES, which is then used to simulate mass and structure data. Assemble the list-structured spread database and serialize it (e.g. as spread_database.rds) so that mzExacto() and decontamination steps can consume it downstream.

## Related tools

- **uafR** (Provides spreadOut(), the function that expands the Unknowns Analysis hit table into uniquely-identified per-datapoint records and stores the list-structured spread database) — github.com/castratton/uafR
- **R** (Runtime environment in which the spreadOut() expansion and annotation-fetch step is executed)
- **webchem** (Supplies get_cid() to convert each tentative Compound.Name into a PubChem CID, from which exact monoisotopic mass and preferred name are fetched)
- **NCI CACTUS** (Fallback chemical resolver generating a canonical SMILES for compounds absent from PubChem, used to simulate mass and structure data)

## Examples

```
input_spread = spreadOut(input_dat)
```

## Evaluation signals

- Schema check: the input CSV contains all six required columns (Component.RT, Component.Area, Base.Peak.MZ, File.Name, Compound.Name, Match.Factor) before expansion
- Uniqueness invariant: every expanded record carries a unique identifier and duplicate peak keys were rejected — number of records equals number of valid datapoints
- Field-preservation check: each expanded record retains component area, tentative identity, match factor, base-peak m/z, retention time, and sample ID (File.Name)
- Annotation completeness: each tentative compound has either a PubChem CID (with exact monoisotopic mass and preferred name) or a CACTUS-generated canonical SMILES
- Round-trip check: the serialized spread database (spread_database.rds) reloads as the list-structured object that mzExacto() accepts without column loss

## Limitations

- Compounds that cannot be found in PubChem and cannot get a canonical SMILES from CACTUS are removed during later decontamination, so peak records for them carry no structural annotation
- uafR has not been tested on Q-TOF, Orbitrap, TimsTOF HT, or Astral instruments or on LC-MS data, and the algorithms do not yet apply to MS2, MS3, or ion mobility + MS2 data
- Researchers should still validate that the compound areas identified by the algorithm make chemical and/or biological sense in their study system

## Evidence

- [readme] Strict CSV column requirements for the expansion input: [evidence span withheld — anchor status: external_doc]
- [readme] spreadOut() consumed by mzExacto() downstream: [evidence span withheld — anchor status: external_doc]
- [intro] Package objective grounding the unique-ID record storage: "automates the sorting and collection of component areas across samples in an experiment while simultaneously storing critical information about every tentative molecule"
- [intro] Public-database annotation fetching rationale: "communicates with public chemistry utilities (including PubChem and the National Cancer Institute) to sort and process the aggregated set of all tentatively identified molecules"
- [readme] Concrete function invocation for the expansion step: [evidence span withheld — anchor status: external_doc]
