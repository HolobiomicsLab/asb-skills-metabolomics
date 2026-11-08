---
name: hypergeometric-pathway-enrichment
description: Use when you have a list of differentially abundant metabolites (e.g.
  selected at p < 0.05 and |log2FC| > 1) and need to identify which metabolic pathways
  are statistically over-represented among them.
when_to_use_negative:
- Input is raw LC-MS peak data (e.g. mzML/NetCDF) rather than an already-curated list
  of differentially abundant metabolites — peak annotation and differential testing
  must happen first.
- You need pathway-level enrichment for features whose abundance differences have
  not yet been established, since hypergeometric testing assumes a pre-defined hit
  set.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PlantMetSuite
  role: Web-based tool that performs hypergeometric pathway over-representation analysis
    on differentially abundant metabolites and visualises the resulting dysregulated
    pathways.
  repo: https://github.com/YaonanLab/PlantMetSuite
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:1792606111507618d1e47a16032ea79cc88022a78947ccecf634e14159293561
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YaonanLab/PlantMetSuite
  tools:
  - PlantMetSuite
  - pandas
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# hypergeometric-pathway-enrichment

## Summary

Run hypergeometric over-representation analysis to map a set of differentially abundant metabolites onto dysregulated pathways, as demonstrated in PlantMetSuite where 35 metabolites were tested against a pathway database to recover 10 significant pathways (p < 0.05).

## When to use

Use when you have a list of differentially abundant metabolites (e.g. selected at p < 0.05 and |log2FC| > 1) and need to identify which metabolic pathways are statistically over-represented among them.

## When NOT to use

- Input is raw LC-MS peak data (e.g. mzML/NetCDF) rather than an already-curated list of differentially abundant metabolites — peak annotation and differential testing must happen first.
- You need pathway-level enrichment for features whose abundance differences have not yet been established, since hypergeometric testing assumes a pre-defined hit set.

## Inputs

- Metabolomics abundance matrix (e.g. pandas DataFrame) with disease vs control groups
- List of differentially abundant metabolites (p < 0.05, |log2FC| > 1)
- Pathway database background mapping metabolites to pathways

## Outputs

- Pathway enrichment results (pathway name, hypergeometric enrichment p-value, significance flag)
- Combined benchmark table benchmark_differential_table.csv with 35 metabolite rows and 10 pathway rows

## How to apply

First select the metabolite input set — in the PlantMetSuite acd5 vs Col-0 case study this was the 35 metabolites flagged as differentially abundant using p < 0.05 and |log2FC| > 1. Then run a hypergeometric over-representation test of that set against the pathway database background, collecting pathways whose enrichment p-value is below 0.05 (the case study reported 10 dysregulated pathways). Assemble metabolite rows (metabolite name, log2FC, direction) and pathway rows (pathway name, enrichment p-value, significance) into a single benchmark table and save it as benchmark_differential_table.csv, validating that it contains exactly 35 metabolite rows and 10 pathway rows with the named metabolites showing the correct directions (e.g. chlorophyll b, D-glucose and proline decreased; glutathione, pyridoxal 5'-phosphate, stearidonic acid and UDP-D-glucose elevated).

## Related tools

- **PlantMetSuite** (Web-based tool that performs hypergeometric pathway over-representation analysis on differentially abundant metabolites and visualises the resulting dysregulated pathways.) — https://github.com/YaonanLab/PlantMetSuite

## Examples

```
results = PlantMetSuite.hypergeometric_enrichment(metabolites=diff_metabolites, background=pathway_db, p_cutoff=0.05); results.to_csv('benchmark_differential_table.csv', index=False)
```

## Evaluation signals

- Number of metabolite rows in the output table equals the expected differential set size (35 in the acd5 case study).
- Number of pathway rows equals the expected count of significant pathways (10 in the acd5 case study).
- Each named metabolite carries the correct direction: chlorophyll b, D-glucose and proline decreased; glutathione, pyridoxal 5'-phosphate, stearidonic acid and UDP-D-glucose elevated.
- All retained pathways satisfy the enrichment threshold p < 0.05.
- Output file benchmark_differential_table.csv exists, parses as valid CSV, and contains the required metabolite/pathway columns with log2FC or enrichment p-value and direction/significance.

## Limitations

- Results depend on the completeness and plant-specificity of the underlying pathway database; the article notes that existing solutions 'lack a dedicated plant-specific metabolite database', which can bias which pathways are detected.
- The hypergeometric test is applied to a fixed hit set; if the differential-abundance filtering step (p < 0.05, |log2FC| > 1) is mis-specified, pathway counts will be wrong.
- As a browser-accessible tool, PlantMetSuite offers no programmable API for batch scripting of this enrichment step.

## Evidence

- [abstract] Existing plant metabolomics solutions lack a dedicated plant-specific metabolite database and pose usability challenges: "Existing solutions lack a dedicated plant-specific metabolite database and pose usability challenges."
- [abstract] PlantMetSuite provides interactive bioinformatics tools and databases tailored to plant metabolomics data: "PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data"
- [abstract] PlantMetSuite is browser-accessible without installation or programming skills: "PlantMetSuite can be accessed directly through a user's browser without the need for installation or programming skills."
- [methods] Hypergeometric over-representation workflow on 35 differentially abundant metabolites producing 10 dysregulated pathways: [evidence span withheld — anchor status: not_found]
