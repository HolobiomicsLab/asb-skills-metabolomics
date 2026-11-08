---
name: molecular-feature-tracking-across-studies
description: Use when a differentially abundant molecular feature from a mouse-model
  untargeted metabolomics study (e.g. a microbially-modulated carnitine in AD serum)
  needs to be matched to public human or animal metabolomics datasets to assess tissue,
  organism, and disease associations.
when_to_use_negative:
- When the query feature already has a fully validated structural annotation from
  a reference library (Level 1 MSI annotation); tissueMASST complements rather than
  replaces library-based annotation.
- When the analysis goal is de novo structure elucidation of a completely unknown
  feature with no MS/MS fragmentation data; tissueMASST requires an MS/MS spectrum
  to search.
- When the feature is expected to be absent from public repositories because of study-specific
  extraction methods, workflows, or instrumentation; lack of detection in tissueMASST
  does not necessarily mean the feature is absent in tissue.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: tissueMASST
  role: Mass spectrometry search tool used to query known or unknown MS/MS spectra
    across public metabolomics repositories, organized by tissue and disease metadata
- name: GNPS
  role: Source of public reference spectral libraries used for MS/MS spectral matching
    and annotation, and the MASST-family search framework tissueMASST builds upon
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:690622d400136dc0a5d53afb6512661118cecaa199e74cb8d1e675e1919bf0bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - tissueMASST
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# molecular-feature-tracking-across-studies

## Summary

Bridges animal-model metabolomics findings to human data by searching known or unknown MS/MS spectra (e.g. microbially-modulated carnitines like phenylacetyl-carnitine) against a reference database of public MS/MS files organized by tissue, organism, and disease metadata via the tissueMASST search tool.

## When to use

Use when a differentially abundant molecular feature from a mouse-model untargeted metabolomics study (e.g. a microbially-modulated carnitine in AD serum) needs to be matched to public human or animal metabolomics datasets to assess tissue, organism, and disease associations.

## When NOT to use

- When the query feature already has a fully validated structural annotation from a reference library (Level 1 MSI annotation); tissueMASST complements rather than replaces library-based annotation.
- When the analysis goal is de novo structure elucidation of a completely unknown feature with no MS/MS fragmentation data; tissueMASST requires an MS/MS spectrum to search.
- When the feature is expected to be absent from public repositories because of study-specific extraction methods, workflows, or instrumentation; lack of detection in tissueMASST does not necessarily mean the feature is absent in tissue.

## Inputs

- MS/MS spectra of query molecular features from the mouse-model untargeted metabolomics dataset (e.g. precursor m/z and fragmentation peaks for benzoyl-carnitine at m/z 266.138, phenylacetyl-carnitine at m/z 280.153, phenylpropionyl-carnitine at m/z 294.169)
- Reference database of public MS/MS files from GNPS/MassIVE, MetaboLights, and Metabolomics Workbench with tissue, organism, and disease-status metadata
- Tissue and disease annotations for the reference files (e.g. 58 tissues, >20 diseases including AD, T2D, IBD)

## Outputs

- tissue_masst_search_results.csv — query-by-query search results with matched reference spectra, source study, organism, tissue, and disease-status metadata
- Per-query hit lists showing tissue and disease distributions of matched features (e.g. phenylacetyl-carnitine mapping to human blood and urine)

## How to apply

Assemble a reference database of public MS/MS files from the three major repositories queried by tissueMASST (GNPS/MassIVE, MetaboLights, Metabolomics Workbench), collecting ~45,000 files tagged with tissue and disease metadata. For each query feature, extract its MS/MS spectrum and search it against the reference database using fragment-peak cosine-similarity spectral matching, as in the GNPS MASST family of tools. Record per-query hit lists containing the source study, organism, tissue, and disease-status metadata for each matching spectrum. Aggregate hits to map the feature's occurrence across tissues and disease conditions; for example, searching phenylacetyl-carnitine returns matches mapping to human blood and urine and reflects enrichment in inflammatory-condition datasets, consistent with Figure 6B of the paper. As a companion to multi-organ murine metabolomics atlases, this step translates animal-model findings (e.g. benzoyl-, phenylacetyl-, phenylpropionyl-carnitine as discriminatory serum features in 3xTg mice) into human AD context.

## Related tools

- **tissueMASST** (Mass spectrometry search tool used to query known or unknown MS/MS spectra across public metabolomics repositories, organized by tissue and disease metadata)
- **GNPS** (Source of public reference spectral libraries used for MS/MS spectral matching and annotation, and the MASST-family search framework tissueMASST builds upon)

## Examples

```
python tissue_masst_search.py --query m/z 280.153 --reference_db 44992_files --metadata tissues_diseases.csv --output tissue_masst_search_results.csv
```

## Evaluation signals

- Searching phenylacetyl-carnitine (m/z 280.153) must return matches mapping to human blood and urine, consistent with Figure 6B of the paper.
- Hits for phenylacetyl-carnitine should reflect enrichment of the feature in inflammatory-condition datasets, consistent with the paper's finding of positive association with aging and cognitive impairment across human AD studies.
- The output file tissue_masst_search_results.csv must be populated with per-query results containing source study, organism, tissue, and disease-status metadata for each match.
- The reference database should reproduce the paper's stated composition of 44,992 public MS/MS files as of April 2025 (27,954 human, 11,422 mouse, 1,328 rat).
- Spectral matches should use fragment-peak cosine-similarity scoring as in the GNPS MASST family, with matched reference spectra carrying valid tissue and disease metadata.

## Limitations

- Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation; lack of detection does not necessarily mean the feature is absent in tissue.
- MS/MS annotations from GNPS reference libraries represent Level 2 or 3 annotations according to the Metabolomics Standards Initiative, not fully validated structures.
- Some spectral pairs may be missing from the cross-tissue molecular network due to poor fragmentation or missing fragments during MS/MS acquisition.

## Evidence

- [abstract] tissueMASST is a mass spectrometry search tool to bridge animal model findings with human data: "Molecular feature tracking via tissueMASST, a mass spectrometry search tool we developed to bridge animal model findings with human data"
- [intro] tissueMASST investigates known or unknown MS/MS spectra across public metabolomics repositories: "we build and introduce tissueMASST, an open-source mass spectrometry (MS) search tool to investigate known or unknown MS/MS spectra across public metabolomics repositories"
- [abstract] phenylacetyl-carnitine is positively associated with aging and cognitive impairment across human AD studies: "identifies microbially-modulated phenylacetyl-carnitine as positively associated with aging and cognitive impairment across human AD studies"
- [discussion] Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods: "Feature detection across reference datasets present in tissueMASST can be influenced by study-specific extraction methods, workflows, and instrumentation. Thus, lack of detection does not necessarily"
