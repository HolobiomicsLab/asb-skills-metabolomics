---
name: csv-dataframe-processing
description: Use when the input is a tabular .csv or .xlsx sample-by-metabolite file
  that must be loaded, parsed, normalized, or reduced to pairwise statistics such
  as a Pearson/Kendall/Spearman correlation matrix.
when_to_use_negative:
- Input is raw instrument output (e.g. unprocessed MS or NMR spectra) rather than
  a preprocessed CSV/Excel table — Viime explicitly picks up the workflow after preprocessing
  and annotation, and annotation is not required but preprocessing is assumed done.
- The required statistical operation is a specialized algorithm implemented in an
  R package (e.g. missForest or impute for imputation) — those run through the OpenCPU
  R service, not pandas dataframe operations.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: pandas
  role: Parses raw CSV/Excel files, normalizes the metabolite table, and computes
    the pairwise Pearson/Kendall/Spearman correlation matrix via DataFrame.corr()
- name: Viime
  role: Hosts the dataframe-processing workflow behind its Flask REST backend, including
    the correlation network edge filtering by minimum threshold
  repo: girder/viime
- name: OpenCPU
  role: Bridges pandas and R data frames for CSV file processing functions implemented
    in the custom R package
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.21105/joss.02410
    title: 'Viime: Visualization and Integration of Metabolomics Experiments'
  source_content_hash: sha256:0140f6e3a9022ba4d74c6c4bc97bd90c55adbb9840754d5c8c54dff97a450d7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/girder/viime
  tools:
  - OpenCPU
  - Viime
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# csv-dataframe-processing

## Summary

Ingest a CSV/Excel metabolomics table into a pandas DataFrame and perform dataframe-level operations (parsing, normalization, pairwise metabolite correlation computation) as the data backbone of Viime's Flask processing backend.

## When to use

Use when the input is a tabular .csv or .xlsx sample-by-metabolite file that must be loaded, parsed, normalized, or reduced to pairwise statistics such as a Pearson/Kendall/Spearman correlation matrix.

## When NOT to use

- Input is raw instrument output (e.g. unprocessed MS or NMR spectra) rather than a preprocessed CSV/Excel table — Viime explicitly picks up the workflow after preprocessing and annotation, and annotation is not required but preprocessing is assumed done.
- The required statistical operation is a specialized algorithm implemented in an R package (e.g. missForest or impute for imputation) — those run through the OpenCPU R service, not pandas dataframe operations.

## Inputs

- CSV or Excel metabolite intensity table (samples x metabolites), .csv or .xlsx
- Configurable minimum correlation threshold and correlation method choice (Pearson, Kendall Tau, or Spearman rank)

## Outputs

- pandas DataFrame of the parsed/normalized metabolite table
- Full pairwise metabolite correlation matrix
- Filtered weighted edge list written to metabolite_correlation_edges.csv with metabolite identifiers and signed correlation weights

## How to apply

Load the processed metabolite intensity table (samples x metabolites) from the input CSV into a pandas DataFrame, since Viime's backend 'leverages Pandas for raw file parsing and normalization'. For correlation networks, compute the full pairwise metabolite correlation matrix by calling pandas' corr() on the metabolite columns with the selected method — Pearson, Kendall Tau, or Spearman rank. Reshape the matrix into pairwise edge records retaining the signed correlation coefficient, then filter edges to keep only pairs whose absolute correlation exceeds the configurable minimum threshold before emitting the weighted edge list. Note that some CSV file processing functions in Viime are exposed through a custom R package served over OpenCPU, where 'the backend code at viime/opencpu.py handles the communication between pandas and R data frames'; pure pandas suffices for correlation computation, while R-backed functions accept a CSV path and return a data frame.

## Related tools

- **pandas** (Parses raw CSV/Excel files, normalizes the metabolite table, and computes the pairwise Pearson/Kendall/Spearman correlation matrix via DataFrame.corr())
- **Viime** (Hosts the dataframe-processing workflow behind its Flask REST backend, including the correlation network edge filtering by minimum threshold) — girder/viime
- **OpenCPU** (Bridges pandas and R data frames for CSV file processing functions implemented in the custom R package)

## Examples

```
import pandas as pd; df = pd.read_csv('metabolite_intensity.csv'); edges = df.corr(method='pearson')
```

## Evaluation signals

- Every emitted edge in metabolite_correlation_edges.csv has an absolute correlation strictly above the configured minimum threshold
- Edge weights carry the correct sign of the correlation (negative in red, positive in gray in the network view)
- All metabolite pairs present in the input table were evaluated in the full pairwise matrix before filtering
- Parsed DataFrame column/row dimensions match the source .csv/.xlsx table (samples x metabolites)
- Selected correlation method (pearson, kendall, spearman) is honored — e.g. Kendall results differ numerically from Pearson on the same data

## Limitations

- R-backed CSV processing functions in viime/opencpu.py are described as work-in-progress, and adding methods requires rebuilding the OpenCPU docker image
- Missing-data handling must be resolved before correlation computation — Viime's imputation heuristically classifies missing data as MNAR or MCAR before analysis
- Correlation edges are only drawn above the configurable minimum threshold, so weak correlations are invisible in the network output

## Evidence

- [abstract] backend uses pandas for parsing and normalization: "The backend leverages Pandas for raw file parsing and normalization."
- [readme] R package CSV processing contract: "Functions contained in this package should accept a path to a CSV file as an argument and return a data frame."
- [readme] pandas-R dataframe bridge: "The (work-in-progress) backend code at [viime/opencpu.py](viime/opencpu.py) handles the communication between pandas and R data frames."
