---
name: msms-spectral-search
description: Use when a molecular feature of interest from an untargeted metabolomics
  dataset is unannotated or only Level 2/3 annotated, and you want to map it to prior
  public MS/MS data from specific tissues, organisms, or disease cohorts.
when_to_use_negative:
- The feature is already annotated to MSI Level 1 via an authentic standard — a tissue
  search adds no structural certainty
- The goal is de novo structure elucidation of a novel molecule with no prior MS/MS
  detection in any public repository
- The input is raw LC-MS/MS data requiring peak detection and feature quantification
  first; run a metabolomics processing pipeline before spectral search
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3336
tools:
- name: tissueMASST
  role: 'Primary tool: performs the MS/MS spectral search against public metabolomics
    repositories and returns matched datasets with tissue/organism/disease metadata'
- name: GNPS
  role: Provides the reference spectral libraries used for Level 2/3 annotation and
    the cosine-similarity spectral-matching framework underlying tissueMASST
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:8e1b5370cc0890437c90581d14bfe98af3dbc41dea90bd701f153171073b3a58
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - tissueMASST
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# msms-spectral-search

## Summary

Search unknown MS/MS spectra against public metabolomics repository reference databases (e.g. via tissueMASST) to identify where a feature has been detected before, linking animal-model findings to human cohorts and assigning tissue, organism, and disease metadata to molecular features.

## When to use

Use when a molecular feature of interest from an untargeted metabolomics dataset is unannotated or only Level 2/3 annotated, and you want to map it to prior public MS/MS data from specific tissues, organisms, or disease cohorts.

## When NOT to use

- The feature is already annotated to MSI Level 1 via an authentic standard — a tissue search adds no structural certainty
- The goal is de novo structure elucidation of a novel molecule with no prior MS/MS detection in any public repository
- The input is raw LC-MS/MS data requiring peak detection and feature quantification first; run a metabolomics processing pipeline before spectral search

## Inputs

- MS/MS spectrum of a molecular feature (precursor m/z and fragment peak list) from an untargeted metabolomics dataset, e.g. from mouse serum or brain
- Optional: list of features of interest (e.g. microbially-modulated carnitines such as benzoyl-, phenylacetyl- and phenylpropionyl-carnitine)
- Optional: metadata table linking features to study, organism, tissue, and disease status for annotation of search results

## Outputs

- tissue_masst_search_results.csv — query-by-query search results recording source study, organism, tissue, and disease-status metadata for each MS/MS match
- Tissue and disease distribution summaries of matched reference spectra per query feature

## How to apply

Take the MS/MS spectrum of the feature (e.g. a precursor m/z with its fragment peaks from a mouse-model or human cohort dataset, such as phenylacetyl-carnitine at m/z 280.153) and submit it as a query to tissueMASST, which searches against ~44,992 public MS/MS files (27,954 human, 11,422 mouse, 1,328 rat as of April 2025) drawn from GNPS/MassIVE, MetaboLights, and Metabolomics Workbench. The tool scores the query against reference spectra using fragment-peak cosine similarity (as in the GNPS MASST family) and aggregates matches into per-query hit lists recording source study, organism, tissue, and disease status. Inspect the returned tissue and disease distributions to contextualize the feature — e.g. phenylacetyl-carnitine hits map to human blood and urine and are enriched in inflammatory-condition datasets (AD, T2D, IBD). Write query-by-query search results to tissue_masst_search_results.csv for downstream trend analysis across human AD cohorts.

## Related tools

- **tissueMASST** (Primary tool: performs the MS/MS spectral search against public metabolomics repositories and returns matched datasets with tissue/organism/disease metadata)
- **GNPS** (Provides the reference spectral libraries used for Level 2/3 annotation and the cosine-similarity spectral-matching framework underlying tissueMASST)

## Examples

```
from tissue_masst import tissue_search; hits = tissue_search(spectrum='PAC_mz280.153.mgf', output='tissue_masst_search_results.csv')
```

## Evaluation signals

- Searching phenylacetyl-carnitine (m/z 280.153) and phenylpropionyl-carnitine (m/z 294.169) must return matches mapping to human blood and urine
- Hit lists for phenylacetyl-carnitine should reflect enrichment in inflammatory-condition datasets (e.g. AD, T2D, IBD), consistent with Figure 6B of the paper
- Output CSV (tissue_masst_search_results.csv) must contain, for each query, the source study, organism, tissue, and disease-status metadata of every matched reference file
- Aggregated tissue distribution across hits should cover the tissue and disease categories described in the reference database (58 tissues; >20 diseases)
- Reference database composition should match the stated April 2025 snapshot: 44,992 public MS/MS files, 27,954 human, 11,422 mouse, 1,328 rat

## Limitations

- Feature detection across reference datasets is influenced by study-specific extraction methods, workflows, and instrumentation — lack of detection in a repository does not mean the feature is absent from that tissue
- Annotations obtained via spectral matching against public GNPS reference libraries are only Level 2 or 3 (MSI), not fully validated structures
- Some true spectral pairs may be missed due to poor fragmentation or missing fragments during MS/MS acquisition
- It remains unclear whether conjugation of microbial phenylalanine-derived precursors into carnitines is microbe- or host-mediated; spectral search does not resolve this

## Evidence

- [intro] tissueMASST, an open-source mass spectrometry (MS) search tool to investigate known or unknown MS/MS spectra across public metabolomics repositories: "we build and introduce tissueMASST, an open-source mass spectrometry (MS) search tool to investigate known or unknown MS/MS spectra across public metabolomics repositories"
- [results] tissueMASST reference database composition (44,992 public MS/MS files; human/mouse/rat breakdown): [evidence span withheld — anchor status: not_found]
- [intro] Phenylacetyl-carnitine maps to human AD cohorts and is positively associated with aging and cognitive impairment: "we mapped phenylacetyl-carnitine to multiple human AD cohorts, where this metabolite was positively associated with aging and cognitive impairment"
- [discussion] Annotation level caveat for GNPS spectral matching: "MS/MS spectra annotation via spectral matching against the public GNPS reference libraries represent a Level 2 or 3 annotation"
- [discussion] Feature-detection caveat across reference datasets: "Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation"
