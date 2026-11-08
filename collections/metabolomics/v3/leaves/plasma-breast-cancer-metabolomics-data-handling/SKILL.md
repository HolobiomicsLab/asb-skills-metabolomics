---
name: plasma-breast-cancer-metabolomics-data-handling
description: Use when input is a metabolite-level abundance matrix (e.g. Lilikoi's
  plasma_breast_cancer.csv) and the goal is Lilikoi's dimension transformation from
  metabolite features to pathway-level PDS features.
when_to_use_negative:
- Input is already a pathway-level feature table (e.g. an existing PDS matrix) — re-running
  the dimension transformation would be redundant; proceed directly to lilikoi.featuresSelection
  or classification.
- The analysis goal is prognosis modeling only (lilikoi.prognosis with Cox-PH/Cox-nnet)
  on an existing expression matrix, which does not require the metabolite-to-pathway
  transformation.
- The dataset lacks recognizable metabolite name identifiers that lilikoi.MetaTOpathway('name')
  can map to HMDB ids and pathways.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3174
tools:
- name: Lilikoi v2.0
  role: Provides lilikoi.Loaddata, lilikoi.MetaTOpathway, and lilikoi.PDSfun used
    to load plasma_breast_cancer.csv, map metabolites to pathways, and compute the
    PDS matrix
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which the Lilikoi v2.0 metabolomics analysis workflow
    is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:4c6caa167185e692606ffdf24d43f4b21662021ae2386529403e96613efde0be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  - lilikoi
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- breast-cancer-metabolomics-data-handling
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# plasma-breast-cancer-metabolomics-data-handling

## Summary

Handles the built-in plasma_breast_cancer.csv metabolomics abundance matrix shipped with Lilikoi v2.0, converting raw metabolite-level measurements into a per-sample pathway dysregulation score (PDS) matrix via lilikoi.MetaTOpathway and lilikoi.PDSfun.

## When to use

Use when input is a metabolite-level abundance matrix (e.g. Lilikoi's plasma_breast_cancer.csv) and the goal is Lilikoi's dimension transformation from metabolite features to pathway-level PDS features.

## When NOT to use

- Input is already a pathway-level feature table (e.g. an existing PDS matrix) — re-running the dimension transformation would be redundant; proceed directly to lilikoi.featuresSelection or classification.
- The analysis goal is prognosis modeling only (lilikoi.prognosis with Cox-PH/Cox-nnet) on an existing expression matrix, which does not require the metabolite-to-pathway transformation.
- The dataset lacks recognizable metabolite name identifiers that lilikoi.MetaTOpathway('name') can map to HMDB ids and pathways.

## Inputs

- plasma_breast_cancer.csv metabolomics abundance matrix bundled in the lilikoi R package (extdata)
- Metadata table with sample labels returned by lilikoi.Loaddata
- Metabolite names as identifiers ('name' type) for HMDB/pathway mapping

## Outputs

- Metabolite_pathway_table: metabolite-to-pathway mapping table from lilikoi.MetaTOpathway
- PDSmatrix: samples x pathways matrix with all dysregulation scores normalized to the [0,1] interval
- pds_matrix.csv: serialized PDS matrix (one row per sample, one column per pathway)

## How to apply

Load the example data with lilikoi.Loaddata pointing at system.file('extdata', 'plasma_breast_cancer.csv', package = 'lilikoi'), which returns a list with $Metadata (including the phenotype Label column) and $dataSet (the metabolite abundance matrix). Run convertResults = lilikoi.MetaTOpathway('name') to map each metabolite name in the dataset to its associated metabolic pathway and obtain Metabolite_pathway_table = convertResults$table. Then compute PDSmatrix = lilikoi.PDSfun(Metabolite_pathway_table), which projects the mapped metabolite measurements onto principal curves within each pathway to yield a pathway dysregulation score DP(i) in [0,1] for every sample-pathway pair. Assemble the scores into a PDS matrix with samples as rows and pathways as columns (all entries normalized to [0,1]) and write it to pds_matrix.csv; this matrix is the input Lilikoi's downstream feature-selection (lilikoi.featuresSelection) and classification modules expect.

## Related tools

- **Lilikoi v2.0** (Provides lilikoi.Loaddata, lilikoi.MetaTOpathway, and lilikoi.PDSfun used to load plasma_breast_cancer.csv, map metabolites to pathways, and compute the PDS matrix) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which the Lilikoi v2.0 metabolomics analysis workflow is executed)

## Examples

```
dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi")); convertResults=lilikoi.MetaTOpathway('name'); PDSmatrix=lilikoi.PDSfun(convertResults$table)
```

## Evaluation signals

- pds_matrix.csv exists and parses as a matrix with one row per sample and one column per pathway
- Every score entry in the PDS matrix falls within the [0,1] range (larger PDS values indicate higher dysregulation)
- Metabolite_pathway_table is non-empty and covers the metabolite names present in plasma_breast_cancer.csv
- PDSmatrix row count equals the number of samples in the loaded Metadata/dataSet from lilikoi.Loaddata
- PDSmatrix feeds lilikoi.featuresSelection without dimension errors (e.g. threshold=0.50, method='gain' runs on its columns)

## Limitations

- The PDS transformation depends on metabolite names being mappable to pathways via MetaTOpathway; unmappable metabolite names will drop out of the pathway projection
- The GigaDB accession/DOI for code and data snapshots is referenced as [42] but not resolved in the article sections provided
- The discussion section text was not present in the extracted article content, so no post-hoc caveats from the authors on this step are available

## Evidence

- [readme] Loaddata + built-in dataset: "dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"))"
- [readme] PDS computation: "PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)"
- [intro] dimension transformer module: "It has 4 modules: feature mapper, dimension transformer, feature selector, and classification predictor"
- [intro] pathway-based representation as Lilikoi's distinguishing feature: "Different from other metabolomics analysis packages, the personalized and pathway-based representation of metabolomics features is the highlight of Lilikoi version 1 (v1)."
