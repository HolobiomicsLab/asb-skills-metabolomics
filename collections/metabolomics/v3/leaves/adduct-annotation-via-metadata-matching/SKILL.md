---
name: adduct-annotation-via-metadata-matching
description: Use when building an MSn spectral library from Orbitrap ID-X acquisitions
  and each file must be linked to its compound identity (adduct, in-source fragment)
  via a metadata table keyed on a unique sample ID.
when_to_use_negative:
- When data were acquired without a metadata sheet containing unique sample IDs —
  the substring match cannot link files to compounds
- When structures are already supplied per record and adducts are known a priori —
  direct annotation without name-to-structure lookup suffices
- When processing single-compound standard runs rather than the plate-based 384-well
  batch layout this matching was designed for
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzmine
  role: Imports the raw/mzML data and performs the metadata-based matching that annotates
    compounds by exact mass with expected adducts and in-source fragments via the
    unique sample ID substring
  repo: https://github.com/mzmine/mzmine
- name: Orbitrap ID-X
  role: Instrument whose Xcalibur sequence embeds the unique sample ID in acquired
    files so metadata matching can succeed
- name: Prefect 2
  role: Runs the metadata_cleanup_prefect.py flow that cleans and validates the metadata
    before matching
  repo: https://github.com/corinnabrungs/msn_tree_library
- name: prepare_wikidata_lotus_prefect.py
  role: Updates the LOTUS natural-product lookup file used when resolving compound
    records during metadata cleanup
  repo: https://github.com/corinnabrungs/msn_tree_library
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-025-02813-0
    title: 'MSnLib: efficient generation of open multi-stage fragmentation mass spectral
      libraries'
  source_content_hash: sha256:5bf0a2e1750c58bcb2e82e6f939f6381194d972a1f485460e77188c404d1889e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - Orbitrap ID-X
  - Prefect 2
  - mzmine
  - prepare_wikidata_lotus_prefect.py
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# adduct-annotation-via-metadata-matching

## Summary

Annotate MSn precursors with expected adducts and in-source fragments by matching each data file to a cleaned metadata record through a unique sample ID substring, as done in the MSnLib/mzmine library-building workflow.

## When to use

Use when building an MSn spectral library from Orbitrap ID-X acquisitions and each file must be linked to its compound identity (adduct, in-source fragment) via a metadata table keyed on a unique sample ID.

## When NOT to use

- When data were acquired without a metadata sheet containing unique sample IDs — the substring match cannot link files to compounds
- When structures are already supplied per record and adducts are known a priori — direct annotation without name-to-structure lookup suffices
- When processing single-compound standard runs rather than the plate-based 384-well batch layout this matching was designed for

## Inputs

- Metadata spreadsheet (template-formatted) with unique sample ID, plate_id, and vial/well location columns
- Orbitrap ID-X raw files (or mzML) whose names contain the unique sample ID substring
- Compound structure/identity records; compound names for PubChem Name search when structures are missing

## Outputs

- Annotated precursor entries in mzmine with expected adduct and in-source fragment labels
- Metadata-linked feature lists feeding MSn tree construction and library export (.json, .mgf, .msp)

## How to apply

Prepare a metadata spreadsheet following the shared template so that column names and the minimum required fields are consistent; each row must carry a unique sample ID, a plate name (plate_id batch identifier), and the vial or well location. Acquire data with the Orbitrap ID-X sequence generated for Xcalibur so that raw file names embed the unique sample ID. In mzmine, after import and MSn denoising, match each imported file to its metadata record by finding the metadata sample ID as a substring of the file name, then assign the expected adducts and in-source fragments listed for that record. If a record lacks structure information, resolve the compound name via PubChem Name search before annotation; databases that need local files or special access (e.g. DrugBank) should be set to False in jobs.py when unavailable. The annotated adducts and fragments then drive subsequent MSn tree building and consensus-spectrum merging (SAME_ENERGY, ALL_ENERGIES, ALL_MSN_TO_PSEUDO_MS2).

## Related tools

- **mzmine** (Imports the raw/mzML data and performs the metadata-based matching that annotates compounds by exact mass with expected adducts and in-source fragments via the unique sample ID substring) — https://github.com/mzmine/mzmine
- **Orbitrap ID-X** (Instrument whose Xcalibur sequence embeds the unique sample ID in acquired files so metadata matching can succeed)
- **Prefect 2** (Runs the metadata_cleanup_prefect.py flow that cleans and validates the metadata before matching) — https://github.com/corinnabrungs/msn_tree_library
- **prepare_wikidata_lotus_prefect.py** (Updates the LOTUS natural-product lookup file used when resolving compound records during metadata cleanup) — https://github.com/corinnabrungs/msn_tree_library

## Examples

```
prefect deploy metadata_cleanup_prefect:cleanup_file --name local-deploy --pool local-work
```

## Evaluation signals

- Every imported file resolves to exactly one metadata record via the unique sample ID substring — unmatched files indicate missing metadata rows or ID mismatch
- Annotated precursor neutral masses are consistent with the exact mass plus the expected adduct/in-source fragment deltas listed in the metadata
- Exported library (.json, .mgf, .msp) preserves the adduct and fragment annotations carried through the MSn trees
- Records lacking structure information have been resolved via PubChem Name search or flagged (e.g. databases set to False in jobs.py) rather than silently skipped
- Well-plate visualization (e.g. 384-well pie charts) reflects the metadata plate_id and well-location fields

## Limitations

- External databases such as DrugBank or Dictionary of Natural Products need local files and/or special access; if unavailable they must be set to False in jobs.py, reducing annotation coverage
- If no structure information is provided, identity depends entirely on PubChem Name search, which can fail for ambiguous or non-standard names
- Chimeric or impure precursor isolations (flagged by isolation purity scoring) can carry incorrect adduct/fragment associations if not filtered before annotation

## Evidence

- [methods] Annotate compounds by exact mass with expected adducts/in-source fragments matched via unique sample ID substring: [evidence span withheld — anchor status: not_found]
- [readme] Metadata template with shared column names: "Please use the [template]...for your metadata for having same column names and minimum needed information for the query."
- [readme] PubChem fallback for missing structures: "If no structure information is provided, it is queried from PubChem by Name search."
- [readme] Databases needing local access are toggled off: "some need a local file and/or special access otherwise set it to False in the jobs.py"
- [methods] Sequence creation embeds unique sample ID and plate metadata: "The sequence creation is setup for the Orbitrap ID-X (Xcalibur). The metadatasheet needs to hava a unique sample id, a plate name (batch identifier) as plate_id and the vial or well location as"
- [methods] Well visualization shows per-well compound mixtures: "The example shows a 384 well plate and each well contains 10 different compounds."
