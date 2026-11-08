---
name: msms-spectral-library-matching
description: Use when chemical annotations must be assigned to MS/MS spectra in public
  MassIVE files or one's own data by spectral reference matching (GNPS library search),
  or when building a binary annotation-by-file matrix from ReDU's pre-computed results.
when_to_use_negative:
- Input is already a annotated feature table or the goal is de novo / molecular-networking
  based annotation — molecular networking is a separate GNPS workflow with its own
  3000-file limit, not library search
- Spectra are in proprietary vendor formats that have not been converted to .mzXML/.mzML,
  since the annotation workflow requires open-format converted data uploaded to GNPS/MassIVE
- Level 1 confirmed identifications (e.g. authentic standard verification) are required
  — spectral reference matching only reaches level 2 or level 3 confidence
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GNPS
  role: Executes the spectral library search (MS/MS spectral library matching) workflow;
    annotations are performed in GNPS by comparing MS2 spectra with reference MS2
    fragmentation patterns
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: ReDU
  role: Launchpad that assembles public MassIVE files via the File Selector, supplies
    the batch template (metabatchdump) and pre-computes library search results for
    the Chemical Explorer
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: MassIVE
  role: Public data repository hosting the raw/converted files and the ReDU identification
    database (MSV000084206) used for library search; ReDU only interacts with MassIVE
- name: Emperor
  role: Visualizes PCA results computed on the chemicals annotated in each file by
    library search
  repo: https://github.com/biocore/emperor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:273443ce372986acee7b75a30e7e1b8450ddc8fa068053755f63ab50afe229d4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mwang87/ReDU-MS2-Documentation
  repo_url: https://github.com/CCMS-UCSD/GNPSDocumentation
  tools:
  - Emperor
  - GNPS
  - MassIVE
  - ReDU
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# msms-spectral-library-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate compounds in tandem MS data by matching MS/MS (product ion) spectra against curated reference spectral libraries in GNPS, as pre-computed at repository scale in ReDU.

## When to use

Use when chemical annotations must be assigned to MS/MS spectra in public MassIVE files or one's own data by spectral reference matching (GNPS library search), or when building a binary annotation-by-file matrix from ReDU's pre-computed results.

## When NOT to use

- Input is already a annotated feature table or the goal is de novo / molecular-networking based annotation — molecular networking is a separate GNPS workflow with its own 3000-file limit, not library search
- Spectra are in proprietary vendor formats that have not been converted to .mzXML/.mzML, since the annotation workflow requires open-format converted data uploaded to GNPS/MassIVE
- Level 1 confirmed identifications (e.g. authentic standard verification) are required — spectral reference matching only reaches level 2 or level 3 confidence

## Inputs

- MS/MS (MS2, product ion) spectra from tandem mass spectrometry data, preferably converted to open formats (.mzXML or .mzML)
- Public MassIVE files with ReDU sample information (metadata), assembled into file cohorts via the ReDU File Selector
- Reference MS2 spectral libraries (e.g. ReDU identification database, MassIVE accession MSV000084206; NIST-style reference spectra)

## Outputs

- Chemical annotations per file: spectral library matches to named compounds (e.g. 'Spectral Match to <Compound> from NIST14')
- Binary annotation-by-file matrix (1 = detected, 0 = not detected) such as gnps_library_search_annotations.tsv
- GNPS library search job result TSVs (batch task outputs for each file batch)

## How to apply

Perform chemical annotation via GNPS spectral library search (MS/MS spectral library matching): compare each acquired MS2 (product ion) spectrum with reference MS2 fragmentation patterns stored in curated libraries (e.g. the ReDU identification database, MassIVE accession MSV000084206) and accept matches whose m/z and abundance patterns are sufficiently similar. Respect GNPS's recommended limit of 5000 files per library search job (3000 for molecular networking), so batch large repositories by splitting file sets. Expect one chemical to yield multiple GNPS annotations because slight variation in MS2 spectra (m/z or abundance) causes the pattern to match different reference MS2 spectra; retain all annotations rather than collapsing them. Treat resulting annotations as level 2 (putative annotation based on spectral library similarity) or level 3 of the 2007 metabolomics standards initiative — not confirmed identifications. For ReDU-style repository re-analysis, download the GNPS batch template (redu.ucsd.edu/metabatchdump), run the Batch Workflow for Spectral Library Search on all matched files, and assemble the returned per-task TSVs into an annotation-versus-file matrix coded 1 (detected) / 0 (not detected).

## Related tools

- **GNPS** (Executes the spectral library search (MS/MS spectral library matching) workflow; annotations are performed in GNPS by comparing MS2 spectra with reference MS2 fragmentation patterns) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **ReDU** (Launchpad that assembles public MassIVE files via the File Selector, supplies the batch template (metabatchdump) and pre-computes library search results for the Chemical Explorer) — https://github.com/mwang87/ReDU-MS2-GNPS
- **MassIVE** (Public data repository hosting the raw/converted files and the ReDU identification database (MSV000084206) used for library search; ReDU only interacts with MassIVE)
- **Emperor** (Visualizes PCA results computed on the chemicals annotated in each file by library search) — https://github.com/biocore/emperor

## Examples

```
wget https://redu.ucsd.edu/metabatchdump && gnps_batch_workflow --workflow SpectralLibrarySearch --batch_batch_file batchdump.tsv
```

## Evaluation signals

- Matrix rows correspond to the compounds enumerated in the ReDU compoundslist (https://redu.ucsd.edu/compoundslist) and columns to the public files reported by ReDU
- Spot-check per-file annotation counts and compound identities against the 'All Data in ReDU' view of the ReDU Chemical Explorer
- Each chemical may legitimately appear as multiple distinct annotations (e.g. 'Spectral Match to Sulfachloropyridazine from NIST14') — do not treat duplicates as errors
- Batch size does not exceed 5000 files per GNPS library search job (3000 for molecular networking)
- A compound-enrichment dashboard URL such as /compoundenrichmentdashboard?compound=Spectral%20Match%20to%20Sulfachloropyridazine%20from%20NIST14 loads and renders functional association graphs

## Limitations

- GNPS annotations via spectral reference matching are only level 2 (putative, spectral library similarity) or level 3 of the 2007 metabolomics standards initiative — not confirmed structures
- The same chemical can receive multiple GNPS annotations due to slight m/z or abundance variation in MS2 spectra, so the annotation list is redundant by design
- Recommended file limits (5000 for library search, 3000 for molecular networking) require batching; very large file sets cannot be searched in one job
- Recently contributed ReDU data are not displayed immediately — data are updated periodically, so new files may be missing from library search results

## Evidence

- [abstract] Chemical annotation is performed in GNPS by comparing MS2 spectra, specifically product ion spectra, with reference MS2 fragmentation patterns: [evidence span withheld — anchor status: not_found]
- [readme] Run Batch Workflow for Spectral Library Search: [evidence span withheld — anchor status: external_doc]
- [methods] matrix cells coded 1 (detected) or 0 (not detected): [evidence span withheld — anchor status: not_found]
