---
name: public-repository-data-harvesting-massive
description: Use when a repository-scale chemical annotation table over public tandem
  MS files is needed, i.e., when a downstream analysis requires a compound-by-file
  binary matrix rather than running new GNPS library search jobs.
when_to_use_negative:
- When the goal is novel chemical annotation of a private dataset not yet deposited
  in MassIVE — run your own GNPS spectral library search instead of harvesting ReDU's
  pre-computed results
- When a finer-grained quantity is needed than binary detection (e.g., precursor intensities
  or cosine scores) — the ReDU matrix only encodes annotated/not-annotated per compound
  per file
- When molecular networking co-analysis is intended — that is a separate ReDU/GNPS
  analysis with its own 3000-file recommended limit, not the library-search harvest
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ReDU
  role: Source of the pre-computed annotations and the compoundslist; the Chemical
    Explorer's 'All Data in ReDU' view is the validation reference
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: GNPS
  role: Platform whose pre-computed MS/MS spectral library search results over public
    MassIVE files are being harvested; library search jobs are capped at 5000 files
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: MassIVE
  role: Public data repository holding the ReDU identification database (MSV000084206)
    and the public files covered by the annotations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:92ce9beb2c2f12e6a09abda7be810e8a4339ecb42e6ba4647bcaddf089f1e5bd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mwang87/ReDU-MS2-Documentation
  repo_url: https://github.com/mwang87/ReDU-MS2-GNPS
  tools:
  - GNPS
  - MassIVE
  - ReDU
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# public-repository-data-harvesting-massive

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harvests ReDU's pre-computed GNPS spectral library search annotations for all public MassIVE files and encodes them as a binary chemical-annotation-by-file matrix (1 = detected, 0 = not detected), preserving multiple GNPS annotations per chemical.

## When to use

Use when a repository-scale chemical annotation table over public tandem MS files is needed, i.e., when a downstream analysis requires a compound-by-file binary matrix rather than running new GNPS library search jobs.

## When NOT to use

- When the goal is novel chemical annotation of a private dataset not yet deposited in MassIVE — run your own GNPS spectral library search instead of harvesting ReDU's pre-computed results
- When a finer-grained quantity is needed than binary detection (e.g., precursor intensities or cosine scores) — the ReDU matrix only encodes annotated/not-annotated per compound per file
- When molecular networking co-analysis is intended — that is a separate ReDU/GNPS analysis with its own 3000-file recommended limit, not the library-search harvest

## Inputs

- ReDU Chemical Explorer compoundslist endpoint (https://redu.ucsd.edu/compoundslist) enumerating all annotated compounds
- ReDU identification database in GNPS/MassIVE (accession MSV000084206)
- Pre-computed GNPS spectral library search results for all public MassIVE files

## Outputs

- gnps_library_search_annotations.tsv: binary chemical-annotation-by-file matrix with cells coded 1 (detected) or 0 (not detected)
- Per-file annotation lists retaining multiple GNPS annotations per chemical

## How to apply

Download the published chemical annotation list served by the ReDU Chemical Explorer at https://redu.ucsd.edu/compoundslist, which enumerates all compounds annotated across the ReDU corpus, and note the ReDU identification database accession MSV000084206 in MassIVE. Retrieve the pre-computed GNPS MS/MS spectral library search results that ReDU has run for all public MassIVE files using default library search parameters, respecting the GNPS recommended limit of 5000 files per library search job. Parse the results to extract file-level annotations, retaining the multiple GNPS annotations a single chemical can receive due to slight variation in MS2 spectra (m/z or abundance) that cause the pattern to match different reference MS2 spectra. Build a binary annotation-by-file matrix (rows = compounds, columns = public files, cells = 1 detected / 0 not detected) and write it as gnps_library_search_annotations.tsv. Validate by spot-checking per-file annotation counts and compound identities against the 'All Data in ReDU' view of the ReDU Chemical Explorer; matrix rows must correspond to compoundslist entries and columns to the public files reported by ReDU.

## Related tools

- **ReDU** (Source of the pre-computed annotations and the compoundslist; the Chemical Explorer's 'All Data in ReDU' view is the validation reference) — https://github.com/mwang87/ReDU-MS2-GNPS
- **GNPS** (Platform whose pre-computed MS/MS spectral library search results over public MassIVE files are being harvested; library search jobs are capped at 5000 files) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **MassIVE** (Public data repository holding the ReDU identification database (MSV000084206) and the public files covered by the annotations)

## Examples

```
curl -L https://redu.ucsd.edu/compoundslist -o compoundslist.tsv && python -c "import pandas as pd; df=pd.read_csv('gnps_library_search_annotations.tsv',sep='\t'); assert df.set_index('compound').applymap(lambda v: v in (0,1)).all().all()"
```

## Evaluation signals

- Matrix rows correspond one-to-one to entries in the compoundslist downloaded from https://redu.ucsd.edu/compoundslist
- Matrix columns correspond to the public files reported by ReDU
- Spot-check of per-file annotation counts matches the 'All Data in ReDU' view of the ReDU Chemical Explorer
- Cell values are strictly binary (1 = detected, 0 = not detected) with no missing or out-of-range encodings
- Compounds with multiple GNPS annotations (from slight m/z or abundance variation in MS2 spectra) are retained rather than collapsed

## Limitations

- GNPS annotations via spectral reference matching are only level 2 or level 3 by the 2007 metabolomics standards initiative, so the matrix is putative annotation, not confirmed identification
- The same chemical can receive multiple GNPS annotations because slight variation in the MS2 spectra (m/z or abundance) matches different reference MS2 spectra
- ReDU data are updated periodically, so recently contributed data may not be reflected in the harvested annotations
- ReDU only interacts with MassIVE, so data in other repositories (e.g., MetaboLights) are out of scope for this harvest

## Evidence

- [other] The library search results are displayed on GNPS and can be downloaded as a table of every chemical annotation versus the files it was detected in: [evidence span withheld — anchor status: not_found]
