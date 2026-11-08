---
name: covid-19-multi-omics-data-handling
description: Use when the analysis target is the Su et al. 2020 COVID-19 metabolomics
  + proteomics abundance dataset (127 samples, mild vs moderate/severe labels) and
  pathway-level features are needed for predictive modelling.
when_to_use_negative:
- The input dataset is not Su et al. 2020 COVID-19 multi-omics and lacks metabolite/protein
  identifier mappings to ChEBI/UniProt needed for Reactome multi-omics pathways
- Samples lack measurements for all omics layers — PathIntegrate requires all input
  omics measured on the same individuals and discards incomplete rows
- The goal is conventional differential-abundance analysis (ORA/GSEA on the molecular
  table) rather than building pathway-level predictive features
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: sspa
  role: Loads/cleans the COVID example metabolomics data, downloads multi-omics Reactome
    pathways, and computes single-sample pathway scores (sspa_SVD / PLAGE) with min_entity=2
  repo: https://github.com/cwieder/py-ssPA
- name: PathIntegrate
  role: Consumes the pathway-transformed COVID-19 multi-omics matrix, automatically
    standardises scores, and fits Single-View or Multi-View models
  repo: https://github.com/cwieder/PathIntegrate
- name: PathIntegrate_scripts
  role: Contains the COVID_data folder with Su et al. 2020 metabolomics and proteomics
    data and replication scripts for the case study
  repo: cwieder/PathIntegrate_scripts
- name: SciKitLearn
  role: Provides the predictive models (e.g. RandomForestClassifier) trained on the
    pathway score matrix, with ssPA transformers compatible via fit/transform and
    Pipeline
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:b94d8a0705669e3b028e050a12d1e9d8ccaf96ef121be19332b6b4149f4b99eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/py-ssPA
  tools:
  - PathIntegrate
  - PathIntegrate_scripts
  - Python
  - SciKitLearn
  - sspa
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# covid-19-multi-omics-data-handling

## Summary

Handles the Su et al. 2020 COVID-19 multi-omics dataset (metabolomics and proteomics, 127 samples, mild vs moderate/severe) for pathway-based integration with PathIntegrate, including identifier mapping, log-transformation, and single-sample pathway scoring via the sspa package.

## When to use

Use when the analysis target is the Su et al. 2020 COVID-19 metabolomics + proteomics abundance dataset (127 samples, mild vs moderate/severe labels) and pathway-level features are needed for predictive modelling.

## When NOT to use

- The input dataset is not Su et al. 2020 COVID-19 multi-omics and lacks metabolite/protein identifier mappings to ChEBI/UniProt needed for Reactome multi-omics pathways
- Samples lack measurements for all omics layers — PathIntegrate requires all input omics measured on the same individuals and discards incomplete rows
- The goal is conventional differential-abundance analysis (ORA/GSEA on the molecular table) rather than building pathway-level predictive features

## Inputs

- Su et al. 2020 COVID-19 metabolomics abundance matrix (samples x metabolites, ChEBI IDs, 127 samples)
- Su et al. 2020 COVID-19 proteomics abundance matrix (samples x proteins, UniProt IDs)
- Outcome labels: mild vs moderate/severe COVID-19 status per sample
- Reactome multi-omics pathway definitions (ChEBI, UniProt, gene_symbol identifiers) via sspa.process_reactome

## Outputs

- Multi-omics single-sample pathway score matrix (samples x 732 Reactome pathway scores, min_entity=2)
- Cleaned, log2-transformed, z-scored abundance matrices suitable for ssPA transformation
- Paired outcome label vector aligned to the pathway score matrix rows

## How to apply

Load the Su et al. 2020 metabolomics and proteomics abundance matrices as pandas DataFrames with samples as rows and molecule IDs as columns; the sspa package provides the processed metabolomics version with ChEBI identifiers as column names via sspa.load_example_data(omicstype='metabolomics', processed=True). Pre-process by removing metabolites with >50% missing/zero values, median-imputing, log2-transforming, and z-score standardising (mean=0, std=1) as shown in the py-ssPA README quickstart. Download multi-omics Reactome pathways with sspa.process_reactome('Homo sapiens', omics_type='multiomics', identifiers=['chebi', 'uniprot', 'gene_symbol']) so metabolites (ChEBI) and proteins (UniProt) map into shared pathway definitions. Compute multi-omics single-sample pathway scores with the SVD-based PLAGE method (sspa.sspa_SVD), retaining pathways with at least 2 measured molecules (min_entity=2), which yields 732 multi-omics pathways on this dataset; PathIntegrate then standardises the pathway score matrix automatically. The resulting pathway score matrix plus COVID/non-COVID severity labels feed Single-View (random forest, AUC ~0.95) or Multi-View (MB-PLS) models.

## Related tools

- **sspa** (Loads/cleans the COVID example metabolomics data, downloads multi-omics Reactome pathways, and computes single-sample pathway scores (sspa_SVD / PLAGE) with min_entity=2) — https://github.com/cwieder/py-ssPA
- **PathIntegrate** (Consumes the pathway-transformed COVID-19 multi-omics matrix, automatically standardises scores, and fits Single-View or Multi-View models) — https://github.com/cwieder/PathIntegrate
- **PathIntegrate_scripts** (Contains the COVID_data folder with Su et al. 2020 metabolomics and proteomics data and replication scripts for the case study) — cwieder/PathIntegrate_scripts
- **SciKitLearn** (Provides the predictive models (e.g. RandomForestClassifier) trained on the pathway score matrix, with ssPA transformers compatible via fit/transform and Pipeline)

## Examples

```
reactome_pathways = sspa.process_reactome(organism="Homo sapiens"); scores = sspa.sspa_SVD(reactome_pathways, min_entity=2).fit_transform(covid_data.iloc[:, :-2])
```

## Evaluation signals

- The pathway score matrix retains exactly 732 multi-omics Reactome pathways when filtering with min_entity=2 on the Su et al. 2020 data
- Random-forest Single-View model on the pathway scores reaches test-set AUC ~0.95, vs 0.98 for the concatenated molecular-level baseline
- RFECV on the tuned random forest selects exactly 20 pathways (e.g. Interleukin-5/13 signalling)
- Pre-processed metabolomics matrix is log2-transformed and z-scored (mean=0, std=1 per column) with ChEBI identifiers as column names
- Every retained pathway has at least 2 measured molecules and pathway coverage per pathway is recorded

## Limitations

- PathIntegrate requires all input omics measured on the same individuals; samples from individuals without data on all omics must be discarded
- Pathway coverage (proportion of pathway molecules observable in the data) is an inherent bottleneck, particularly for metabolomics
- Un-annotated molecules cannot be incorporated into pathway-based models, causing information loss
- Comparing molecular-level vs pathway-level models is challenging because the number of differentially abundant molecules per pathway is unknown and pathway definitions vary between databases

## Evidence

- [readme] sspa provides a Python interface for metabolomics pathway analysis: [evidence span withheld — anchor status: external_doc]
- [readme] load example data: [evidence span withheld — anchor status: external_doc]
- [readme] pre-processing quickstart: "# Remove metabolites with too many NA values
data_filt = covid_values.loc[:, covid_values.isin([' ', np.nan, 0]).mean() < 0.5]"
- [readme] Su et al. data availability: [evidence span withheld — anchor status: external_doc]
- [abstract] pathway-level transformation: "PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level"
- [discussion] dataset application: "We applied PathIntegrate to two datasets: COPDgene and COVID19 multi-omics."
- [readme] multi-omics pathways: [evidence span withheld — anchor status: external_doc]
