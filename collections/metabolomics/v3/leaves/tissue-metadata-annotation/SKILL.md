---
name: tissue-metadata-annotation
description: 'Use when building or extending the tissueMASST reference index: public
  MS/MS files must be tagged with tissue-associated metadata (organism, tissue, disease
  status) before spectral search queries can be mapped to biological context.'
when_to_use_negative:
- When the goal is to search a query spectrum against existing tissueMASST data (spectral
  search), not to build or extend the underlying tissue-annotated reference database.
- When input MS/MS files already carry complete tissue-associated metadata and only
  need spectral scoring — annotation is done; proceed directly to scoring.
- When the reference data is not public MS/MS from the three supported repositories
  (e.g., proprietary or non-public datasets); tissueMASST's annotation pipeline assumes
  MassIVE/MetaboLights/Workbench provenance.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: tissueMASST
  role: Hosts the tissue-annotated MS/MS reference database; this skill supplies the
    metadata annotation layer that tissueMASST search relies on to return organism,
    tissue, and disease-status for each spectral match.
- name: GNPS
  role: Provides the public MS/MS repositories (MassIVE) and the spectral-matching
    framework that tissueMASST's reference data are drawn from.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:e18229c431a9599d0145eb1b0a5a85ac1ba5dff808c5f04315ca4ea01bb1a5e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - tissueMASST
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# tissue-metadata-annotation

## Summary

Assemble and annotate a tissue-aware MS/MS reference database for tissueMASST by collecting public MS/MS files from GNPS/MassIVE, MetaboLights, and Metabolomics Workbench and attaching tissue-associated metadata so query spectra can be matched back to organism, tissue, and disease-status context.

## When to use

Use when building or extending the tissueMASST reference index: public MS/MS files must be tagged with tissue-associated metadata (organism, tissue, disease status) before spectral search queries can be mapped to biological context.

## When NOT to use

- When the goal is to search a query spectrum against existing tissueMASST data (spectral search), not to build or extend the underlying tissue-annotated reference database.
- When input MS/MS files already carry complete tissue-associated metadata and only need spectral scoring — annotation is done; proceed directly to scoring.
- When the reference data is not public MS/MS from the three supported repositories (e.g., proprietary or non-public datasets); tissueMASST's annotation pipeline assumes MassIVE/MetaboLights/Workbench provenance.

## Inputs

- Public MS/MS files from GNPS/MassIVE, MetaboLights, and Metabolomics Workbench (44,992 files as of April 2025)
- Study-level metadata (organism, tissue, disease status) from each contributing repository submission
- Reference peak lists indexed by precursor m/z

## Outputs

- Tissue-annotated MS/MS reference database (27,954 human, 11,422 mouse, 1,328 rat files)
- Metadata-enriched index keyed by precursor m/z covering 58 tissues and >20 diseases (e.g., AD, T2D, IBD)

## How to apply

Collect public MS/MS files from the three repositories used by tissueMASST (GNPS/MassIVE, MetaboLights, Metabolomics Workbench), targeting the paper's stated reference composition of 44,992 public MS/MS files as of April 2025 (27,954 human, 11,422 mouse, 1,328 rat). Parse each file with pandas and attach tissue-associated metadata covering the 58 tissues and more than 20 diseases described (e.g., AD, T2D, IBD), including organism, tissue, disease status, and source study accession. Index the annotated MS/MS peak lists by precursor m/z to enable fast candidate lookup during spectral search. Note that feature detection in these reference datasets is influenced by study-specific extraction methods, workflows, and instrumentation, so metadata completeness directly determines how reliable downstream tissue/disease interpretation is.

## Related tools

- **tissueMASST** (Hosts the tissue-annotated MS/MS reference database; this skill supplies the metadata annotation layer that tissueMASST search relies on to return organism, tissue, and disease-status for each spectral match.)
- **GNPS** (Provides the public MS/MS repositories (MassIVE) and the spectral-matching framework that tissueMASST's reference data are drawn from.)

## Evaluation signals

- Reference file count matches the paper's stated composition: 44,992 total, 27,954 human, 11,422 mouse, 1,328 rat.
- Every indexed reference file has non-null tissue-associated metadata (organism, tissue, disease status where applicable), covering the 58 tissues and >20 diseases described.
- Spot-check spectral matches returned for known query spectra (e.g., phenylacetyl-carnitine m/z 280.153) correctly map to human blood and urine per Figure 6B of the paper.
- Metadata fields are populated from repository submission records rather than inferred — verify provenance for a random sample of entries.
- Indexing by precursor m/z returns candidate hits for a benchmark query within expected latency (no unindexed full-scan).

## Limitations

- Feature detection in reference datasets is confounded by study-specific extraction methods, workflows, and instrumentation — absence of detection does not mean absence in tissue.
- Metadata completeness varies across repository submissions; missing tissue/disease labels reduce downstream interpretation quality.
- The reference database reflects a snapshot (April 2025); newly deposited public MS/MS files are not included until re-annotation.

## Evidence

- [other] tissueMASST's reference database comprises 44,992 public MS/MS files as of April 2025, of which 27,954 were acquired from human tissues, 11,422 from mice, and 1,328 from rats.: [evidence span withheld — anchor status: not_found]
- [discussion] Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation. Thus, lack of detection does not necessarily: "Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation. Thus, lack of detection does not necessarily"
- [intro] we build and introduce tissueMASST, an open-source mass spectrometry (MS) search tool to investigate known or unknown MS/MS spectra across public metabolomics repositories: "we build and introduce tissueMASST, an open-source mass spectrometry (MS) search tool to investigate known or unknown MS/MS spectra across public metabolomics repositories"
