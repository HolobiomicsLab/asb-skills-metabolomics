---
name: ms2-spectral-database-curation-characterization
description: Use when a local annotation tool ships a built-in MS2 spectral library
  (e.g. Met-ID's FMP-10 database) and you must quantify its contents — number of derivatized
  forms, parent standards, CID energy range — before trusting downstream identifications.
when_to_use_negative:
- Input is user-acquired MS2 spectra from unknown tissue compounds meant for similarity
  comparison against the library, not characterization of the library itself
- The analysis targets underivatized/conventional-matrix MALDI-MSI data whose expected
  adducts are [M+H]+ or [M-H]- rather than FMP-10 derivatization products
- You intend to add user-generated spectra to the database — this skill only characterizes
  the preconfigured bundled entries, and README notes all database changes affect
  only the local install
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Met-ID
  role: Provides the MetIDServer Python API used to start the local server, enumerate
    database tables, and load metabolite/MS2 spectral entries for counting and characterization
  repo: https://github.com/pbjarterot/Met-ID
- name: pandas
  role: Writes the extracted database metrics (derivatized_forms, parent_standards,
    cid_min_eV, cid_max_eV) to the characterization CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:9d4437138d95018a126bb8a5065fb38875baf47787108886f24db5a7716ee992
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pbjarterot/Met-ID
  tools:
  - Met-ID
  - pandas
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# ms2-spectral-database-curation-characterization

## Summary

Characterize and curate a bundled MS2 spectral reference database (e.g. Met-ID's FMP-10 library of derivatized neurotransmitter/metabolite standards) by enumerating its derivatized spectral forms, distinct parent chemical standards, and collision-energy coverage, then validating those metrics against article-reported values.

## When to use

Use when a local annotation tool ships a built-in MS2 spectral library (e.g. Met-ID's FMP-10 database) and you must quantify its contents — number of derivatized forms, parent standards, CID energy range — before trusting downstream identifications.

## When NOT to use

- Input is user-acquired MS2 spectra from unknown tissue compounds meant for similarity comparison against the library, not characterization of the library itself
- The analysis targets underivatized/conventional-matrix MALDI-MSI data whose expected adducts are [M+H]+ or [M-H]- rather than FMP-10 derivatization products
- You intend to add user-generated spectra to the database — this skill only characterizes the preconfigured bundled entries, and README notes all database changes affect only the local install

## Inputs

- Locally installed Met-ID software with its bundled FMP-10 MS2 spectral database (SQLite tables)
- MetIDServer client API (MetIDServer context manager, list_tables(), list_metabolites(), list_matrices(), list_functional_groups(), list_adducts())
- Article-reported reference metrics for validation (154 derivatized forms, 66 chemical standards, CID 0–50 eV)

## Outputs

- fmp10_ms2_database_characterization.csv with columns derivatized_forms, parent_standards, cid_min_eV, cid_max_eV
- Consistency verdict on whether the installed database build matches the reported release (154 forms / 66 standards / 0–50 eV)

## How to apply

Start the local Met-ID server using the MetIDServer context manager so the bundled SQLite-backed spectral database is reachable, then enumerate its tables with list_tables() and load the MS2 spectral/metabolite tables via client convenience methods such as list_metabolites(). Count distinct derivatized spectral forms in the FMP-10 database, taking care to account for multiple derivatization sites per molecule (Met-ID's key feature for reactive matrices like FMP-10), and count distinct parent chemical standards via unique accession/name fields. Extract the collision-energy span of the library entries — the reference library was acquired with a 1 Da isolation window at collision energies 0–50 eV on an FT-ICR instrument. Write the extracted metrics (derivatized_forms, parent_standards, cid_min_eV, cid_max_eV) to a CSV with pandas. Validate by comparing counts to the article-reported release: 154 derivatized forms from 66 chemical standards spanning CID 0–50 eV; any mismatch flags the installed database build as inconsistent (note the README warns that reinstalling Met-ID does not always update bundled database files, so stale libraries are a real failure mode).

## Related tools

- **Met-ID** (Provides the MetIDServer Python API used to start the local server, enumerate database tables, and load metabolite/MS2 spectral entries for counting and characterization) — https://github.com/pbjarterot/Met-ID
- **pandas** (Writes the extracted database metrics (derivatized_forms, parent_standards, cid_min_eV, cid_max_eV) to the characterization CSV)

## Examples

```
from metid import MetIDServer; with MetIDServer() as server: print(server.list_tables()); mets = server.list_metabolites()
```

## Evaluation signals

- Extracted derivatized_forms equals 154 (article-reported value for the FMP-10 MS2 database)
- Extracted parent_standards equals 66 distinct chemical standards
- CID energy range spans exactly 0 to 50 eV (cid_min_eV=0, cid_max_eV=50), consistent with the 1 Da isolation-window FT-ICR acquisition reported in the paper
- list_tables() returns the expected metabolite/MS2 spectral tables, confirming the MetIDServer context manager started the bundled database rather than an empty or stale install
- Any mismatch between extracted metrics and the reported release flags the database build as inconsistent and triggers the documented fix of manually removing the 'com.farmbio.metid' application-support folder and reinstalling

## Limitations

- The database holds a preconfigured finite number of MS2 spectra; no automated MS2 collection for MSI is available (noted as future work)
- Reinstalling newer versions of Met-ID does not update bundled database files, which must be removed manually (the 'com.farmbio.metid' folder in AppData/Application Support)
- All database changes (e.g. added user spectra) affect only the specific local install, not other installations
- Version numbers for Met-ID and dependencies such as rdkit are not stated in the article

## Evidence

- [abstract] Met-ID includes a database featuring MS2 spectra of numerous chemical standards, consisting of neurotransmitters and metabolites derivatized with FMP-10: "Met-ID includes a database featuring MS2 spectra of numerous chemical standards, consisting of neurotransmitters and metabolites derivatized with FMP-10"
- [discussion] The MS2 analyses used a 1 Da isolation window and spectra collected at different collision energies (0–50 eV): "The MS2 analyses used a 1 Da isolation window and spectra collected at different collision energies (0–50 eV)"
- [methods] Import MetIDServer and use it as a context manager.: "Import ``MetIDServer`` and use it as a context manager."
- [readme] The base version of Met-ID comes with a number of MS2 spectra collected from chemical standards with an FT-ICR using the FMP-10 chemical matrix.: [evidence span withheld — anchor status: external_doc]
- [discussion] some limitations exist, particularly regarding the software’s preconfigured finite number of MS2 spectra: "some limitations exist, particularly regarding the software’s preconfigured finite number of MS2 spectra"
