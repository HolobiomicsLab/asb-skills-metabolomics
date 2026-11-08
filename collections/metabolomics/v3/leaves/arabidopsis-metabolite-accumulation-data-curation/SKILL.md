---
name: arabidopsis-metabolite-accumulation-data-curation
description: Use when building or reproducing a metadata-annotated metabolomics MOG
  project from Arabidopsis GC-MS metabolite accumulation data and mutation metadata
  hosted on PMR, e.g. the 656-sample, 50-mutant AT-metab-dataset.
when_to_use_negative:
- The input is not metabolite accumulation data with accompanying sample metadata
  (e.g. it is raw GC-MS instrument output not yet processed into a metabolite-by-sample
  table)
- The goal is analysis of the AT-microarray-dataset or Hu-cancer-RNASeq-dataset case
  studies rather than the Arabidopsis metabolomics dataset
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaOmGraph
  role: Import the PMR metabolite accumulation table and metadata, link metadata to
    sample columns, and save the integrated project as AT-Mutation-242-656-metab.mog
  repo: https://github.com/urmi-21/MetaOmGraph/
- name: MOG
  role: Standalone Java workbench used to assemble and store the curated, metadata-annotated
    metabolomics project
  repo: http://metnetweb.gdcb.iastate.edu/MetNet_MetaOmGraph.htm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz1209
    title: 'MetaOmGraph: a workbench for interactive exploratory data analysis of
      large expression datasets'
  source_content_hash: sha256:0c34964885040ca4531b7578e6bdde8a99bb0f6b17bc20e687920bf04bad6485
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/urmi-21/MetaOmGraph
  tools:
  - MOG
  - MetaOmGraph
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# arabidopsis-metabolite-accumulation-data-curation

## Summary

Reproduces the MetaOmGraph (MOG) project AT-Mutation-242-656-metab.mog by curating the AT-metab-dataset GC-MS metabolite accumulation data for 50 A. thaliana knockout/knockdown mutants (656 samples) retrieved from PMR, integrating sample metadata, and saving the annotated dataset as a savable/shareable MOG project file.

## When to use

Use when building or reproducing a metadata-annotated metabolomics MOG project from Arabidopsis GC-MS metabolite accumulation data and mutation metadata hosted on PMR, e.g. the 656-sample, 50-mutant AT-metab-dataset.

## When NOT to use

- The input is not metabolite accumulation data with accompanying sample metadata (e.g. it is raw GC-MS instrument output not yet processed into a metabolite-by-sample table)
- The goal is analysis of the AT-microarray-dataset or Hu-cancer-RNASeq-dataset case studies rather than the Arabidopsis metabolomics dataset

## Inputs

- AT-metab-dataset GC-MS metabolite accumulation data table for 50 A. thaliana mutants (from PMR)
- Sample metadata table describing knockout/knockdown mutation conditions (from PMR)

## Outputs

- AT-Mutation-242-656-metab.mog — a MetaOmGraph project file containing the 656-sample metabolomics dataset with linked mutation metadata
- Integrated, metadata-annotated metabolomics dataset with 656 samples spanning 50 mutations

## How to apply

Download the AT-metab-dataset GC-MS metabolite accumulation table for the 50 A. thaliana knockout/knockdown mutants from the PMR resource, along with the accompanying sample metadata describing the mutation conditions (the paper notes 'Data and metadata were retrieved from PMR'). Launch MetaOmGraph (Java .jar requiring JRE 8+; optional R 3.4+), import the metabolite data table, and link the sample metadata to the data columns so each of the 656 samples is annotated with its mutation condition. Verify the assembled project contains 656 samples spanning the 50 mutations before saving. Save the integrated dataset as a MetaOmGraph project file named AT-Mutation-242-656-metab.mog; MOG projects preserve the history of explorations and can be shared, reused, and included in publications.

## Related tools

- **MetaOmGraph** (Import the PMR metabolite accumulation table and metadata, link metadata to sample columns, and save the integrated project as AT-Mutation-242-656-metab.mog) — https://github.com/urmi-21/MetaOmGraph/
- **MOG** (Standalone Java workbench used to assemble and store the curated, metadata-annotated metabolomics project) — http://metnetweb.gdcb.iastate.edu/MetNet_MetaOmGraph.htm

## Examples

```
git clone https://github.com/urmi-21/MetaOmGraph.git
```

## Evaluation signals

- The saved .mog project contains exactly 656 samples corresponding to the 50 knockout/knockdown mutations of A. thaliana genes
- Sample metadata rows are correctly linked to the matching metabolite data columns (mutation condition annotations resolve for every sample column)
- The project file re-opens in MetaOmGraph with the metadata-integrated dataset intact, since MOG projects can be saved and shared
- Data and metadata provenance trace back to the PMR resource as reported in the article

## Limitations

- MOG requires Java Runtime Environment 8 (or higher); R 3.4+ is optional and only needed for sending data subsets to R
- The metabolite data must already be a metabolite-accumulation table with accompanying metadata from PMR; MOG does not process raw GC-MS runs
- MOG is designed for expression-type data explored on local machines; very large datasets rely on its indexing and buffering schemes

## Evidence

- [results] Here, we use the AT-metab-dataset metabolomics dataset that represents a comprehensive study of 50 mutants: "Here, we use the AT-metab-dataset metabolomics dataset that represents a comprehensive study of 50 mutants"
- [results] Data and metadata were retrieved from PMR (61).: "Data and metadata were retrieved from PMR (61)."
- [abstract] MOG projects, with history of explorations, can be saved and shared.: "MOG projects, with history of explorations, can be saved and shared."
- [readme] Java Runtime Environment 8 (or higher); R 3.4 (or higher) [optional]: "* Java Runtime Environment 8 (or higher)
* R 3.4 (or higher) [optional]"
