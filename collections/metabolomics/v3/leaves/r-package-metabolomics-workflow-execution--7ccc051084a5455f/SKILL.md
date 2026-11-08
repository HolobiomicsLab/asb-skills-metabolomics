---
name: r-package-metabolomics-workflow-execution
description: Use when running a personalized pathway-based metabolomics classification
  or prognosis pipeline in R on metabolite-level data that must be converted to pathway-level
  features.
when_to_use_negative:
- The task only requires metabolite-level analysis without pathway-based transformation
  — Lilikoi's highlight is personalized pathway-based representation of metabolomics
  features
- A web-based metabolomics workflow is required; Lilikoi v2.0 is specifically an R
  programming environment package intended to overcome web-application limitations
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Lilikoi v2.0
  role: 'R package providing the full workflow: feature mapper, dimension transformer
    (PDS), feature selector (information gain, threshold 0.50), classification predictor,
    and prognosis modules'
  repo: https://github.com/lanagarmire/lilikoi2
- name: Cox-nnet
  role: Deep learning-based model used within lilikoi.prognosis for prognosis prediction
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which the Lilikoi v2.0 workflow is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:403e1e66ae447e7b6992dc7dccdfc619b6fc2b9f047574c10b5c728ddd3417d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Cox-nnet
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# r-package-metabolomics-workflow-execution

## Summary

Execute the Lilikoi v2.0 end-to-end metabolomics analysis workflow in R, which chains feature mapping, PDS-based dimension transformation, information-gain pathway feature selection, machine/deep learning classification, and prognosis modeling.

## When to use

Use when running a personalized pathway-based metabolomics classification or prognosis pipeline in R on metabolite-level data that must be converted to pathway-level features.

## When NOT to use

- The task only requires metabolite-level analysis without pathway-based transformation — Lilikoi's highlight is personalized pathway-based representation of metabolomics features
- A web-based metabolomics workflow is required; Lilikoi v2.0 is specifically an R programming environment package intended to overcome web-application limitations

## Inputs

- Metabolomics data table (CSV, e.g., plasma_breast_cancer.csv) loadable by lilikoi.Loaddata, yielding Metadata and dataSet
- Metabolite names or HMDB ids for metabolite-to-pathway mapping
- Phenotype labels (e.g., Metadata$Label) for classification; event/time variables for prognosis

## Outputs

- Metabolite-pathway mapping table (Metabolite_pathway_table)
- Pathway dysregulation score (PDS) matrix
- Selected pathway list (selected_Pathways_Weka)
- Classification performance from machine learning (Rpart, LDA, SVM, RF, GBM, LOG, DL)
- Prognosis model results from Cox-PH / Cox-nnet
- KEGG pathway visualization plots

## How to apply

Load the metabolomics dataset with lilikoi.Loaddata (e.g., plasma_breast_cancer.csv) to obtain Metadata and dataSet. Map metabolite names to HMDB ids with lilikoi.MetaTOpathway('name'), then compute the pathway dysregulation score (PDS) matrix via lilikoi.PDSfun using the pathtracer algorithm. Select the most phenotype-relevant pathways from the PDS matrix with lilikoi.featuresSelection using the information-gain method and a 0.50 threshold, exporting selected_Pathways_Weka. Feed the selected pathways into lilikoi.machine_learning (with options such as Rpart, LDA, SVM, RF, GBM, LOG, and a deep-learning DL classifier with dlround=50, 10-fold CV, and 0.8 train portion) or into lilikoi.prognosis using Cox-PH and/or the deep learning-based Cox-nnet model. Optionally add metabolite-pathway regression (lilikoi.meta_path) and KEGG pathway visualization (lilikoi.KEGGplot).

## Related tools

- **Lilikoi v2.0** (R package providing the full workflow: feature mapper, dimension transformer (PDS), feature selector (information gain, threshold 0.50), classification predictor, and prognosis modules) — https://github.com/lanagarmire/lilikoi2
- **Cox-nnet** (Deep learning-based model used within lilikoi.prognosis for prognosis prediction) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which the Lilikoi v2.0 workflow is executed)

## Examples

```
devtools::install_github('lanagarmire/lilikoi2'); dt <- lilikoi.Loaddata(file=system.file('extdata','plasma_breast_cancer.csv',package='lilikoi')); selected_Pathways_Weka <- lilikoi.featuresSelection(lilikoi.PDSfun(lilikoi.MetaTOpathway('name')$table), threshold=0.50, method='gain')
```

## Evaluation signals

- lilikoi.featuresSelection returns a non-empty selected_Pathways_Weka object — pathways scoring above the 0.50 information-gain threshold are retained
- PDS matrix produced by lilikoi.PDSfun has pathway-level rows/columns derived from the Metabolite_pathway_table mapping, not metabolite-level features
- lilikoi.machine_learning runs with 10-fold cross-validation (cvnum=10), 0.8 training portion, and 10 runs (nrun=10), producing metrics for each enabled classifier (Rpart, LDA, SVM, RF, GBM, LOG, DL)
- lilikoi.prognosis completes with nfold=5 cross-validated lambda selection (cvlambda) and the specified coxnnet setting
- Installation check: package loads from CRAN (install.packages('lilikoi')) or dev version via devtools::install_github('lanagarmire/lilikoi2')

## Limitations

- Prognosis modeling with Cox-nnet may require a Python path (python.path) to be configured
- Code and data snapshots rely on the GitHub repository and GigaDB; the article does not provide the GigaDB accession/DOI in the extracted sections
- Feature selection threshold (0.50) and method ('gain') are fixed defaults in the documented example; other settings are not validated in the provided context

## Evidence

- [intro] It has 4 modules: feature mapper, dimension transformer, feature selector, and classification predictor: "It has 4 modules: feature mapper, dimension transformer, feature selector, and classification predictor"
- [readme] selected_Pathways_Weka= lilikoi.featuresSelection(PDSmatrix,threshold= 0.50,method="gain"): "selected_Pathways_Weka= lilikoi.featuresSelection(PDSmatrix,threshold= 0.50,method="gain")"
- [readme] PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table): "# Transform metabolites into pathway using pathtracer algorithm
PDSmatrix=lilikoi.PDSfun(Metabolite_pathway_table)"
- [readme] prognosis prediction, implemented by Cox-PH model and the deep-learning based Cox-nnet model: "It also has several new modules, including the most significant addition of prognosis prediction, implemented by Cox-PH model and the deep-learning based Cox-nnet model."
