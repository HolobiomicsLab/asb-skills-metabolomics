---
name: public-metabolomics-repository-querying
description: Use when you have an MS/MS feature from an animal model (e.g. phenylacetyl-carnitine
  at m/z 280.153) and want to find which public human/mouse/rat datasets detect it,
  across which tissues and disease states.
when_to_use_negative:
- Input is already a feature-abundance table or you only need a spectral library annotation
  — this skill returns tissue/disease provenance, not identifications, and annotations
  from spectral matching are only Level 2/3 (MSI), not fully validated structures.
- Absence of a query in reference datasets is used to conclude the feature is not
  present in a tissue — detection across reference datasets is confounded by study-specific
  extraction methods, workflows and instrumentation.
- The task is de novo annotation of unknowns with no public reference spectra available,
  or you need quantitative abundance estimates from the repositories rather than presence/absence
  provenance.
edam_operation: http://edamontology.org/operation_0405
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: tissueMASST
  role: Open-source MS search tool that searches query MS/MS spectra against a reference
    database of ~44,992 public files from three repositories, aggregated by tissue
    and disease metadata.
- name: GNPS
  role: Source of reference spectral libraries used for Level 2/3 spectral matching
    and the MASST search-tool family whose cosine-similarity scoring tissueMASST follows.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:ca98c4e3512843529e7612841da32e66de06f2db1c478c13f167422d56c50251
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - tissueMASST
  edam_operation: http://edamontology.org/operation_0405
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# public-metabolomics-repository-querying

## Summary

Search query MS/MS spectra against reference libraries assembled from public metabolomics repositories (GNPS/MassIVE, MetaboLights, Metabolomics Workbench) using tissueMASST-style cosine-similarity matching, to map animal-model features onto human tissues and disease cohorts.

## When to use

Use when you have an MS/MS feature from an animal model (e.g. phenylacetyl-carnitine at m/z 280.153) and want to find which public human/mouse/rat datasets detect it, across which tissues and disease states.

## When NOT to use

- Input is already a feature-abundance table or you only need a spectral library annotation — this skill returns tissue/disease provenance, not identifications, and annotations from spectral matching are only Level 2/3 (MSI), not fully validated structures.
- Absence of a query in reference datasets is used to conclude the feature is not present in a tissue — detection across reference datasets is confounded by study-specific extraction methods, workflows and instrumentation.
- The task is de novo annotation of unknowns with no public reference spectra available, or you need quantitative abundance estimates from the repositories rather than presence/absence provenance.

## Inputs

- Query MS/MS spectra with precursor m/z and fragment peaks (e.g. GNPS-library spectra of benzoyl-, phenylacetyl- and phenylpropionyl-carnitine)
- Reference MS/MS files harvested from public repositories (GNPS/MassIVE, MetaboLights, Metabolomics Workbench), with per-file tissue/organism/disease metadata

## Outputs

- Per-query hit lists recording source study, organism, tissue and disease status for each matching reference spectrum
- Summaries of tissue and disease distributions across all hits for each query
- tissue_masst_search_results.csv — query-by-query search results written to a CSV file

## How to apply

Assemble a reference database of ~44,992 public MS/MS files from GNPS/MassIVE, MetaboLights and Metabolomics Workbench, covering ~58 tissues and >20 diseases (e.g. AD, T2D, IBD). Parse each file with pandas, attach tissue/organism/disease metadata, and index reference peak lists by precursor m/z for fast candidate lookup. Score query spectra (e.g. GNPS-library spectra of benzoyl-, phenylacetyl- and phenylpropionyl-carnitine at m/z 266.138, 280.153, 294.169) against reference spectra with a fragment-peak cosine-similarity spectral-matching score, as in the GNPS MASST family. Aggregate matches into per-query hit lists recording source study, organism, tissue and disease status, and summarize tissue and disease distributions of the hits.

## Related tools

- **tissueMASST** (Open-source MS search tool that searches query MS/MS spectra against a reference database of ~44,992 public files from three repositories, aggregated by tissue and disease metadata.)
- **GNPS** (Source of reference spectral libraries used for Level 2/3 spectral matching and the MASST search-tool family whose cosine-similarity scoring tissueMASST follows.)

## Examples

```
python search_tissue_masst.py --query spectra.mgf --repos gnps,metabolights,metabolomics_workbench --output tissue_masst_search_results.csv
```

## Evaluation signals

- Query-by-query search results must be written to tissue_masst_search_results.csv.
- Searching phenylacetyl-carnitine and phenylpropionyl-carnitine must return matches mapping to human blood and urine.
- Hit tissue/disease distributions should reflect enrichment of phenylacetyl-carnitine in inflammatory-condition datasets, consistent with Figure 6B of the paper.
- Reference database composition must match the stated counts: 44,992 public MS/MS files as of April 2025, of which 27,954 human, 11,422 mouse, and 1,328 rat.
- Each hit record must carry source study, organism, tissue, and disease-status metadata.

## Limitations

- Feature detection across reference datasets is influenced by study-specific extraction methods, workflows and instrumentation; lack of detection does not necessarily mean absence in a tissue.
- Spectral matching yields only Level 2 or 3 annotations under the Metabolomics Standards Initiative, not fully validated structures.
- Some spectral pairs may be missing from results due to poor fragmentation or missing fragments during MS/MS acquisition.

## Evidence

- [other] tissueMASST reference database: [evidence span withheld — anchor status: not_found]
- [intro] tissueMASST intro: "we build and introduce tissueMASST, an open-source mass spectrometry (MS) search tool to investigate known or unknown MS/MS spectra across public metabolomics repositories"
- [abstract] tissueMASST abstract: "Molecular feature tracking via tissueMASST, a mass spectrometry search tool we developed to bridge animal model findings with human data"
- [results] tissueMASST results: "identifies microbially-modulated phenylacetyl-carnitine as positively associated with aging and cognitive impairment across human AD studies"
- [discussion] tissueMASST limitation: "Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation. Thus, lack of detection does not necessarily"
