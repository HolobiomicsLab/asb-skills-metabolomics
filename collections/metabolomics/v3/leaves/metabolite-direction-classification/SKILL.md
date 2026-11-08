---
name: metabolite-direction-classification
description: Use when you have a metabolite abundance matrix with disease/wild-type
  versus control groups and need to flag each metabolite as differentially abundant
  (increased/decreased) using p < 0.05 and |log2FC| > 1 thresholds before pathway
  enrichment.
when_to_use_negative:
- Input is already a pathway enrichment result table (no raw abundance matrix to compute
  log2FC from)
- The study has more than two groups (e.g. multi-factor designs) — pairwise log2FC/p-value
  direction-calling does not capture the experimental structure
- Metabolites lack replicate-level abundance values, so p-values cannot be computed
  and direction calling would be based on fold-change alone
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: PlantMetSuite
  role: Web-based tool providing interactive differential-abundance analysis and pathway
    enrichment for plant metabolomics data; supports browser-accessible analysis without
    programming skills
  repo: github.com/YaonanLab/PlantMetSuite
- name: pandas
  role: Python library used to load the metabolomics abundance matrix, compute log2
    fold changes and p-values, and assemble the benchmark differential table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:9a7093ff6b973ce70299862b32917fb72555df95f2567790d9490f45171f78a1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PlantMetSuite
  - pandas
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# metabolite-direction-classification

## Summary

Classify metabolites as significantly increased or decreased between case and control groups using log2 fold-change direction and p-value significance, then verify named metabolites match their reported directions. This is the differential-abundance direction-calling step that underpins pathway enrichment in plant metabolomics workflows.

## When to use

Use when you have a metabolite abundance matrix with disease/wild-type versus control groups and need to flag each metabolite as differentially abundant (increased/decreased) using p < 0.05 and |log2FC| > 1 thresholds before pathway enrichment.

## When NOT to use

- Input is already a pathway enrichment result table (no raw abundance matrix to compute log2FC from)
- The study has more than two groups (e.g. multi-factor designs) — pairwise log2FC/p-value direction-calling does not capture the experimental structure
- Metabolites lack replicate-level abundance values, so p-values cannot be computed and direction calling would be based on fold-change alone

## Inputs

- Case-study metabolomics abundance matrix (e.g. wild-type Arabidopsis Col-0 vs ceramide kinase mutant acd5) with disease and control group columns
- Metabolite identifiers with reported abundance directions for benchmark verification (e.g. chlorophyll b, D-glucose, proline, glutathione, pyridoxal 5'-phosphate, stearidonic acid, UDP-D-glucose, acetoacetyl-CoA)

## Outputs

- benchmark_differential_table.csv containing exactly 35 metabolite rows with log2FC and direction (increased/decreased) plus 10 pathway rows with enrichment p-values
- Set of differentially abundant metabolites (p < 0.05, |log2FC| > 1) ready for hypergeometric pathway enrichment

## How to apply

Load the case-study metabolomics abundance matrix (disease vs control groups) into pandas. For each metabolite, compute log2 fold change between groups and the p-value of the abundance difference, then flag metabolites as differentially abundant when p < 0.05 and |log2FC| > 1, recording the direction (increased/decreased). Verify named metabolites match reported directions: acetoacetyl-CoA, chlorophyll b, D-glucose, and proline decreased, while glutathione, pyridoxal 5'-phosphate, stearidonic acid, and UDP-D-glucose elevated in the acd5 mutants. Confirm the differentially abundant set totals 35 entries as reported. Feed the 35 flagged metabolites into hypergeometric over-representation pathway enrichment (expecting 10 dysregulated pathways at p < 0.05). Assemble metabolite rows (metabolite, log2FC, direction) and pathway rows (pathway, enrichment p, significance) into a single benchmark table saved as benchmark_differential_table.csv.

## Related tools

- **PlantMetSuite** (Web-based tool providing interactive differential-abundance analysis and pathway enrichment for plant metabolomics data; supports browser-accessible analysis without programming skills) — github.com/YaonanLab/PlantMetSuite
- **pandas** (Python library used to load the metabolomics abundance matrix, compute log2 fold changes and p-values, and assemble the benchmark differential table)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('abundance_matrix.csv'); df['log2FC'] = df.groupby('group')['value'].transform(lambda x: x.mean()) ; flagged = df[(df.p < 0.05) & (df.log2FC.abs() > 1)]; flagged.to_csv('benchmark_differential_table.csv')"
```

## Evaluation signals

- benchmark_differential_table.csv exists and is valid CSV
- File contains exactly 35 metabolite rows and 10 pathway rows
- Named metabolites appear with correct directions: chlorophyll b, D-glucose, proline, acetoacetyl-CoA decreased; glutathione, pyridoxal 5'-phosphate, stearidonic acid, UDP-D-glucose elevated
- Every retained metabolite satisfies p < 0.05 and |log2FC| > 1
- Downstream hypergeometric enrichment yields 10 dysregulated pathways at p < 0.05 from the 35 flagged metabolites

## Limitations

- Direction classification is pairwise (two groups only); it does not handle multi-factor or time-course designs
- Thresholds p < 0.05 and |log2FC| > 1 are the paper's specific cutoffs; using them uncritically on other datasets may over- or under-call metabolites
- PlantMetSuite is browser-based and requires no programming, which limits scripted/batch reproducibility for large-scale pipelines

## Evidence

- [results] chlorophyll b, D-glucose, and proline were considerably diminished, while glutathione, pyridoxal 5'-phosphate, stearidonic acid, and UDP-D-glucose exhibited significant elevation: [evidence span withheld — anchor status: not_found]
- [abstract] PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data, facilitating upstream-to-downstream analysis in metabolomics and supporting: "PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data, facilitating upstream-to-downstream analysis in metabolomics and supporting"
- [abstract] Existing solutions lack a dedicated plant-specific metabolite database and pose usability challenges.: "Existing solutions lack a dedicated plant-specific metabolite database and pose usability challenges."
- [abstract] PlantMetSuite can be accessed directly through a user's browser without the need for installation or programming skills.: "PlantMetSuite can be accessed directly through a user's browser without the need for installation or programming skills. The tool is freely available and will undergo regular updates and expansions"
