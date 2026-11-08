---
name: fold-change-statistical-testing
description: Use when you have a metabolomics abundance matrix with case/control (disease
  vs wild-type) groups and need to identify differentially abundant metabolites with
  direction and statistical significance, e.g. 35 DA metabolites and 10 dysregulated
  pathways.
when_to_use_negative:
- Input is already a curated list of differentially abundant metabolites and only
  pathway enrichment is needed — go straight to hypergeometric over-representation
  instead.
- The data are raw mass-spectrometry peaks needing peak detection/normalization; differential
  testing must come after preprocessing, not replace it.
- No biological replicates or group labels exist, so fold-change and p-value testing
  between conditions is not defined.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: PlantMetSuite
  role: Web tool that performs the differential-abundance (fold-change + statistical
    testing) analysis and visualisation on plant metabolomics data, e.g. the acd5
    vs Col-0 case study.
  repo: github.com/YaonanLab/PlantMetSuite
- name: pandas
  role: Scripted implementation vehicle for loading the abundance matrix, computing
    log2FC and p-values per metabolite, and exporting the benchmark CSV.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:277a24b4134cdcadec6673fdad9c223448c4f11b2a5053bee53ad9442b216155
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PlantMetSuite
  - pandas
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fold-change-statistical-testing

## Summary

Compute per-metabolite log2 fold changes and abundance-difference p-values between case and control groups, then flag metabolites as differentially abundant (e.g. p < 0.05 and |log2FC| > 1) as performed in PlantMetSuite's acd5 vs Col-0 case study.

## When to use

Use when you have a metabolomics abundance matrix with case/control (disease vs wild-type) groups and need to identify differentially abundant metabolites with direction and statistical significance, e.g. 35 DA metabolites and 10 dysregulated pathways.

## When NOT to use

- Input is already a curated list of differentially abundant metabolites and only pathway enrichment is needed — go straight to hypergeometric over-representation instead.
- The data are raw mass-spectrometry peaks needing peak detection/normalization; differential testing must come after preprocessing, not replace it.
- No biological replicates or group labels exist, so fold-change and p-value testing between conditions is not defined.

## Inputs

- metabolomics abundance matrix (feature table) with case and control sample groups (e.g. acd5 mutant vs Col-0 wild type)
- group/phenotype labels mapping samples to disease vs control conditions

## Outputs

- per-metabolite log2 fold-change and p-value table with direction and significance flags
- set of differentially abundant metabolites (e.g. 35 entries for acd5 vs Col-0) with named metabolites (chlorophyll b, D-glucose, proline, glutathione, pyridoxal 5'-phosphate, stearidonic acid, UDP-D-glucose) and directions
- benchmark_differential_table.csv combining metabolite rows and pathway-enrichment rows

## How to apply

Load the metabolomics abundance matrix into pandas and, per metabolite, compute log2 fold change between the two groups plus a p-value for the abundance difference. Flag a metabolite as differentially abundant when p < 0.05 and |log2FC| > 1, recording the direction (increased/decreased). Verify reported directions: chlorophyll b, D-glucose, proline (and acetoacetyl-CoA) decreased; glutathione, pyridoxal 5'-phosphate, stearidonic acid, UDP-D-glucose elevated. Confirm the DA set totals 35 metabolites in the acd5 vs Col-0 benchmark. Feed the 35 DA metabolites downstream to hypergeometric over-representation pathway enrichment (p < 0.05, yielding 10 dysregulated pathways) and assemble a single benchmark CSV with columns for metabolite/pathway, log2FC or enrichment p, and direction/significance. This skill mirrors PlantMetSuite's browser-based differential-abundance analysis, which is designed so no programming is needed — but the same thresholds are reproducible in a pandas script.

## Related tools

- **PlantMetSuite** (Web tool that performs the differential-abundance (fold-change + statistical testing) analysis and visualisation on plant metabolomics data, e.g. the acd5 vs Col-0 case study.) — github.com/YaonanLab/PlantMetSuite
- **pandas** (Scripted implementation vehicle for loading the abundance matrix, computing log2FC and p-values per metabolite, and exporting the benchmark CSV.)

## Examples

```
import pandas as pd; df = pd.read_csv('abundance.csv'); fc = df.groupby('group')['metabolite'].mean(); log2fc = np.log2(fc['acd5']/fc['Col-0'])
```

## Evaluation signals

- Exactly 35 metabolite rows and 10 pathway rows in benchmark_differential_table.csv.
- Named metabolites match reported directions: chlorophyll b, D-glucose, proline decreased; glutathione, pyridoxal 5'-phosphate, stearidonic acid, UDP-D-glucose elevated.
- All retained metabolites satisfy p < 0.05 and |log2FC| > 1.
- Enriched pathways all show p < 0.05 from the hypergeometric test.
- Output file is valid CSV with metabolite/pathway, log2FC or enrichment p, and direction/significance columns.

## Limitations

- Thresholds (p < 0.05, |log2FC| > 1) and group comparisons are analysis-specific; users must confirm they suit their own study design.
- Statistical testing is meaningful only when the input matrix is preprocessed; PlantMetSuite is positioned as upstream-to-downstream, but raw-peak handling is a separate step.
- No changelog is available for the tool, so version-specific behavior cannot be tracked from the README.

## Evidence

- [methods] compute log2 fold change and p-value thresholds: [evidence span withheld — anchor status: not_found]
- [results] acd5 case-study finding: [evidence span withheld — anchor status: not_found]
- [abstract] PlantMetSuite scope: "PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data, facilitating upstream-to-downstream analysis in metabolomics"
- [abstract] no-programming access: "PlantMetSuite can be accessed directly through a user's browser without the need for installation or programming skills."
