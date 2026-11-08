---
name: plant-metabolomics-benchmark-table-construction
description: Use when validating a metabolomics differential-analysis pipeline against
  a published case study (e.g. wild-type Arabidopsis versus a mutant such as acd5)
  where named metabolites, statistical thresholds (p < 0.
when_to_use_negative:
- The input is already a finished feature/importance table or a pre-computed differential
  results file rather than a raw abundance matrix — re-running fold-change and p-value
  computation would be redundant.
- The goal is a general-purpose exploratory metabolomics analysis in PlantMetSuite
  rather than reproducing a specific published benchmark with fixed thresholds (p
  < 0.05, |log2FC| > 1) and expected row counts.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PlantMetSuite
  role: Web-based metabolomics analysis platform whose case study (wild-type Arabidopsis
    vs acd5) supplies the differential-abundance metabolites and pathway-enrichment
    results that this benchmark table reproduces and validates.
  repo: https://github.com/YaonanLab/PlantMetSuite
- name: pandas
  role: Python library used to load the abundance matrix, compute per-metabolite log2
    fold change and p-value, apply the p < 0.05 and |log2FC| > 1 thresholds, and assemble
    the benchmark CSV.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:ec1384f10dac6b2d6910c399323e39316cd443c00b1e50384429c75af4a89401
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YaonanLab/PlantMetSuite
  tools:
  - PlantMetSuite
  - pandas
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# plant-metabolomics-benchmark-table-construction

## Summary

Constructs a validation benchmark table from a PlantMetSuite metabolomics case study: differential-abundance metabolites (statistically defined by p-value and log2 fold change) plus hypergeometric pathway-enrichment results, assembled into a single CSV with direction-of-change annotations.

## When to use

Use when validating a metabolomics differential-analysis pipeline against a published case study (e.g. wild-type Arabidopsis versus a mutant such as acd5) where named metabolites, statistical thresholds (p < 0.05, |log2FC| > 1), and pathway-enrichment outcomes must be reproduced as a tabular benchmark.

## When NOT to use

- The input is already a finished feature/importance table or a pre-computed differential results file rather than a raw abundance matrix — re-running fold-change and p-value computation would be redundant.
- The goal is a general-purpose exploratory metabolomics analysis in PlantMetSuite rather than reproducing a specific published benchmark with fixed thresholds (p < 0.05, |log2FC| > 1) and expected row counts.

## Inputs

- Case-study metabolomics abundance matrix (samples as columns or rows, metabolites as the other axis, disease vs control group labels, e.g. wild-type Col-0 vs acd5 mutant)
- Group assignment labels defining the two conditions to compare
- Metabolite-to-pathway mapping (pathway database) for over-representation analysis

## Outputs

- benchmark_differential_table.csv containing 35 differentially abundant metabolite rows (metabolite, log2 fold change, p-value, direction increased/decreased) and 10 dysregulated pathway rows (pathway, enrichment p-value, significance)
- Validated list of differentially abundant metabolites matching reported directions for named metabolites (e.g. D-glucose and proline decreased; glutathione and UDP-D-glucose elevated)

## How to apply

Load the case-study metabolomics abundance matrix (disease versus control groups) into pandas and, for each metabolite, compute the log2 fold change between groups and the p-value of the abundance difference, flagging metabolites as differentially abundant when p < 0.05 and |log2FC| > 1 while recording the direction (increased or decreased). Verify the resulting differential set against reported named metabolites and their directions: acetoacetyl-CoA, chlorophyll b, D-glucose and proline should be diminished, while glutathione, pyridoxal 5'-phosphate, stearidonic acid and UDP-D-glucose should be significantly elevated. Confirm the differential metabolite set totals 35 entries, then run hypergeometric over-representation pathway enrichment of those 35 metabolites against the pathway database, retaining pathways with enrichment p < 0.05 (expected 10 dysregulated pathways). Assemble the metabolite rows (metabolite, log2FC, direction) and pathway-enrichment rows (pathway, enrichment p, significance) into a single benchmark table, save it as benchmark_differential_table.csv, and validate row counts, file validity, and the presence of the named metabolites with correct directions.

## Related tools

- **PlantMetSuite** (Web-based metabolomics analysis platform whose case study (wild-type Arabidopsis vs acd5) supplies the differential-abundance metabolites and pathway-enrichment results that this benchmark table reproduces and validates.) — https://github.com/YaonanLab/PlantMetSuite
- **pandas** (Python library used to load the abundance matrix, compute per-metabolite log2 fold change and p-value, apply the p < 0.05 and |log2FC| > 1 thresholds, and assemble the benchmark CSV.)

## Examples

```
df = pd.read_csv('abundance_matrix.csv'); import scipy.stats as ss; p = df.apply(lambda r: ss.ttest_ind(r[disease], r[control]).pvalue, axis=1); fc = df[disease].mean(axis=1) - df[control].mean(axis=1)
```

## Evaluation signals

- benchmark_differential_table.csv exists and parses as valid CSV.
- The table contains exactly 35 metabolite rows and 10 pathway rows.
- Named metabolites appear with the reported directions: acetoacetyl-CoA, chlorophyll b, D-glucose and proline decreased; glutathione, pyridoxal 5'-phosphate, stearidonic acid and UDP-D-glucose elevated.
- Every flagged metabolite satisfies p < 0.05 and |log2FC| > 1; every retained pathway has enrichment p < 0.05.

## Limitations

- Thresholds (p < 0.05, |log2FC| > 1) are case-study-specific; other PlantMetSuite analyses may use different cutoffs, so the benchmark does not generalise without re-derivation.
- The hypergeometric pathway-enrichment step depends on the completeness and annotation coverage of the underlying pathway database; incomplete plant-specific pathway mapping can miss dysregulated pathways.
- No changelog was found for PlantMetSuite, so result reproducibility across tool versions is not documented.

## Evidence

- [abstract] chdir: [evidence span withheld — anchor status: not_found]
- [methods] dir-verify: [evidence span withheld — anchor status: not_found]
