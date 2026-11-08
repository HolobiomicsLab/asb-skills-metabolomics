---
name: fmp10-derivatized-standard-counting
description: Use when auditing or characterizing the Met-ID bundled MS2 spectral library
  (FMP-10 matrix), e.g. to verify release contents, validate a database build against
  reported metrics, or summarize derivatization-site coverage before spectral similarity
  searches.
when_to_use_negative:
- Input is already a characterization report or the metrics CSV — the counting step
  is redundant
- The goal is annotating tissue m/z features or identifying derivatized metabolites
  from an MSI dataset (use Met-ID's identify()/predict_ccs() workflow instead, not
  database auditing)
- The MS2 library of interest is a user-added custom database rather than the bundled
  FMP-10 standard set — reported counts (154/66) would not apply
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Met-ID
  role: Provides the MetIDServer context manager, SQLite-backed database, and client
    browse methods (list_tables, list_metabolites) used to enumerate and count FMP-10
    MS2 spectral entries
  repo: https://github.com/pbjarterot/Met-ID
- name: pandas
  role: Writes the extracted characterization metrics (derivatized_forms, parent_standards,
    cid_min_eV, cid_max_eV) to the output CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c00633
    title: 'Met-ID: An Open-Source Software for Comprehensive Annotation of Multiple
      On-Tissue Chemical Modifications in MALDI-MSI'
  source_content_hash: sha256:769abaa2b2c9b27959268b4d8e9c34a8ed6193f57204580e03fb3ff5f95eab1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pbjarterot/Met-ID
  tools:
  - Met-ID
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# fmp10-derivatized-standard-counting

## Summary

Characterize the built-in FMP-10 MS2 spectral database bundled with Met-ID by counting its derivatized spectral forms and distinct parent chemical standards and extracting the CID collision-energy coverage, providing a reproducible consistency check against the article-reported values (154 forms, 66 standards, 0–50 eV).

## When to use

Use when auditing or characterizing the Met-ID bundled MS2 spectral library (FMP-10 matrix), e.g. to verify release contents, validate a database build against reported metrics, or summarize derivatization-site coverage before spectral similarity searches.

## When NOT to use

- Input is already a characterization report or the metrics CSV — the counting step is redundant
- The goal is annotating tissue m/z features or identifying derivatized metabolites from an MSI dataset (use Met-ID's identify()/predict_ccs() workflow instead, not database auditing)
- The MS2 library of interest is a user-added custom database rather than the bundled FMP-10 standard set — reported counts (154/66) would not apply

## Inputs

- Met-ID bundled MS2 spectral database (SQLite tables for the FMP-10 matrix, reachable via MetIDServer)
- Met-ID client API methods: list_tables(), list_metabolites(), list_matrices(), list_functional_groups(), list_adducts()
- Article-reported reference values (154 derivatized forms, 66 chemical standards, CID 0–50 eV)

## Outputs

- fmp10_ms2_database_characterization.csv with columns derivatized_forms, parent_standards, cid_min_eV, cid_max_eV
- Pass/fail validation flag indicating whether extracted counts match the reported release

## How to apply

Start the local Met-ID server using the MetIDServer context manager so the bundled SQLite-backed database is reachable, then enumerate available tables with list_tables() and load the metabolite/MS2 spectral tables via client convenience methods such as list_metabolites(). Count distinct derivatized spectral forms in the FMP-10 database, being careful to account for multiple derivatization sites per molecule (a key Met-ID capability for the reactive matrix FMP-10), and separately count distinct parent chemical standards underlying those spectra via unique accession/name fields. Extract the CID collision-energy range covered by spectral library entries — the standards were acquired with a 1 Da isolation window at collision energies 0–50 eV. Write the metrics (derivatized_forms, parent_standards, cid_min_eV, cid_max_eV) to a CSV with pandas, and validate: 154 derivatized forms from 66 chemical standards spanning 0–50 eV; any mismatch flags the bundled database build as inconsistent with the reported release. Note the README caveat that reinstalling Met-ID does not always refresh bundled database files, so stale 'com.farmbio.metid' app-data folders can distort counts and must be removed before reinstallation.

## Related tools

- **Met-ID** (Provides the MetIDServer context manager, SQLite-backed database, and client browse methods (list_tables, list_metabolites) used to enumerate and count FMP-10 MS2 spectral entries) — https://github.com/pbjarterot/Met-ID
- **pandas** (Writes the extracted characterization metrics (derivatized_forms, parent_standards, cid_min_eV, cid_max_eV) to the output CSV)

## Examples

```
with MetIDServer() as server: client = server.client; client.list_metabolites(); df.to_csv('fmp10_ms2_database_characterization.csv')
```

## Evaluation signals

- Extracted derivatized_forms equals 154, matching the article-reported count of derivatized spectral forms
- Extracted parent_standards equals 66 distinct chemical standards (neurotransmitters and metabolites derivatized with FMP-10)
- Extracted CID range spans 0 to 50 eV, consistent with reported acquisition parameters
- Validation step flags 'inconsistent' if any metric mismatches, so the CSV should carry a coherent pass/fail state
- Database tables enumerated via list_tables() must be non-empty; zero metabolites suggests stale bundled database files (the com.farmbio.metid app-data issue)

## Limitations

- Met-ID has a preconfigured finite number of MS2 spectra; the library only covers standards collected in-house with FMP-10 on an FT-ICR
- Reinstalling newer versions of Met-ID does not update bundled database files; the 'com.farmbio.metid' folder must be manually removed so the installer adds the correct files
- All database modifications (including user-added spectra) affect only that specific install of Met-ID, so local counts may diverge from the reference 154/66 if the database was customized
- No explicit version numbers are stated for Met-ID or its dependencies, so the characterization is tied to whichever release is installed

## Evidence

- [abstract] Met-ID includes a database featuring MS2 spectra of numerous chemical standards, consisting of neurotransmitters and metabolites derivatized with FMP-10: "Met-ID includes a database featuring MS2 spectra of numerous chemical standards, consisting of neurotransmitters and metabolites derivatized with FMP-10"
- [discussion] MS2 analyses used a 1 Da isolation window and spectra collected at different collision energies 0-50 eV: "The MS2 analyses used a 1 Da isolation window and spectra collected at different collision energies (0–50 eV)"
- [methods] Import MetIDServer and use it as a context manager: "Import ``MetIDServer`` and use it as a context manager."
- [abstract] The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound. This ability is exemplified by FMP-10: "The software is able to identify multiple derivatization sites on the same molecule, facilitating identification of the derivatized compound. This ability is exemplified by FMP-10"
- [readme] reinstalling newer versions of Met-ID does not update the database files it comes bundled with: [evidence span withheld — anchor status: external_doc]
