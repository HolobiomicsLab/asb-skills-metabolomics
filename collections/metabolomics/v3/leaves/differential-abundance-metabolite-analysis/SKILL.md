---
name: differential-abundance-metabolite-analysis
description: Use when you have a metabolomics abundance matrix with disease/control
  (or mutant/wild-type) group labels and need to flag which named metabolites are
  differentially abundant — e.g. acd5 vs Col-0 chlorophyll b, D-glucose, proline decreased;
when_to_use_negative:
- When the input is already a curated list of differential metabolites or a finished
  enrichment table — this skill computes the differential hits and enrichment from
  raw abundance data, not re-analysing them.
- When the abundance matrix lacks group labels or has fewer than two sample groups
  (e.g. single-condition metabolomics) — the log2FC/p-value comparison between groups
  is undefined.
- When you need plant-specific metabolite identification or spectral annotation rather
  than statistical testing of already-identified metabolites — PlantMetSuite's dedicated
  plant metabolite database serves the identification step instead.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: PlantMetSuite
  role: Provides the differential-abundance analysis and downstream pathway over-representation/enrichment
    modules used to derive the metabolite hit list and dysregulated pathways; the
    browser-based tool referenced by the source card's workflow.
  repo: https://github.com/YaonanLab/PlantMetSuite
- name: pandas
  role: Loads the case-study abundance matrix, computes per-metabolite log2 fold change
    and p-values, applies the p < 0.05 and |log2FC| > 1 cutoffs, and assembles the
    metabolite and pathway rows into the final benchmark CSV.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/plants12152880
    title: 'PlantMetSuite: A User-Friendly Web-Based Tool for Metabolomics Analysis
      and Visualisation'
  source_content_hash: sha256:c38141e042b5299c4522eff56c75959eaf314c261ef6ec1d0568f9587abf95ee
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# differential-abundance-metabolite-analysis

## Summary

Identifies metabolites that differ significantly in abundance between two sample groups (e.g. wild-type Arabidopsis vs. the ceramide kinase mutant acd5) by computing log2 fold change and p-values per metabolite, then confirms directionality and feeds the hit list into downstream pathway over-representation analysis. The source workflow is demonstrated with PlantMetSuite plus pandas on a metabolomics abundance matrix.

## When to use

Use when you have a metabolomics abundance matrix with disease/control (or mutant/wild-type) group labels and need to flag which named metabolites are differentially abundant — e.g. acd5 vs Col-0 chlorophyll b, D-glucose, proline decreased; glutathione, pyridoxal 5'-phosphate, stearidonic acid, UDP-D-glucose elevated — and want a benchmark table of metabolite and pathway hits.

## When NOT to use

- When the input is already a curated list of differential metabolites or a finished enrichment table — this skill computes the differential hits and enrichment from raw abundance data, not re-analysing them.
- When the abundance matrix lacks group labels or has fewer than two sample groups (e.g. single-condition metabolomics) — the log2FC/p-value comparison between groups is undefined.
- When you need plant-specific metabolite identification or spectral annotation rather than statistical testing of already-identified metabolites — PlantMetSuite's dedicated plant metabolite database serves the identification step instead.

## Inputs

- Metabolomics abundance matrix (rows = metabolites, columns = samples) with disease/control or mutant/wild-type group labels, e.g. the acd5 vs Col-0 case-study matrix
- Pathway/metabolite mapping database for hypergeometric over-representation enrichment (the pathway database used by PlantMetSuite)

## Outputs

- List of differentially abundant metabolites with log2FC, p-value and direction (expected 35 entries for the acd5 case)
- Pathway over-representation result (expected 10 dysregulated pathways, each with enrichment p < 0.05)
- benchmark_differential_table.csv — a CSV with metabolite/pathway rows, log2FC or enrichment p, and direction/significance columns

## How to apply

Load the case-study abundance matrix (disease vs control groups) into pandas and, for each metabolite, compute the log2 fold change between groups plus the p-value of the abundance difference; flag a metabolite as differentially abundant when p < 0.05 and |log2FC| > 1, recording the direction (increased/decreased). Verify named metabolites against expected directions — acetoacetyl-CoA, chlorophyll b, D-glucose and proline decreased, while glutathione, pyridoxal 5'-phosphate, stearidonic acid and UDP-D-glucose were elevated — and confirm the differential set totals 35 entries as reported for acd5. Feed the 35-hit list into hypergeometric over-representation pathway enrichment against the pathway database (PlantMetSuite), retaining pathways with enrichment p < 0.05 (expected 10 dysregulated pathways). Assemble metabolite rows and pathway rows into a single benchmark table with columns for metabolite/pathway, log2FC or enrichment p, and direction/significance, then save as CSV for validation. The rationale is that the article's differential-abundance and pathway-enrichment modules in PlantMetSuite are exactly the statistical backbone for this group-comparison task, and the pandas pre/post-processing keeps the output in a reusable tabular form.

## Related tools

- **PlantMetSuite** (Provides the differential-abundance analysis and downstream pathway over-representation/enrichment modules used to derive the metabolite hit list and dysregulated pathways; the browser-based tool referenced by the source card's workflow.) — https://github.com/YaonanLab/PlantMetSuite
- **pandas** (Loads the case-study abundance matrix, computes per-metabolite log2 fold change and p-values, applies the p < 0.05 and |log2FC| > 1 cutoffs, and assembles the metabolite and pathway rows into the final benchmark CSV.)

## Examples

```
import pandas as pd; df = pd.read_csv('abundance_matrix.csv'); df['log2FC'] = df['disease_mean'] - df['control_mean']; df['p'] = df.apply(lambda r: scipy.stats.ttest_ind(r['disease'], r['control']).pvalue, axis=1); hits = df[(df['p'] < 0.05) & (df['log2FC'].abs() > 1)]
```

## Evaluation signals

- The differential metabolite set contains exactly 35 entries (as reported for acd5 vs Col-0).
- Named metabolites match reported directions: chlorophyll b, D-glucose, proline (and acetoacetyl-CoA) decreased; glutathione, pyridoxal 5'-phosphate, stearidonic acid, UDP-D-glucose elevated.
- The pathway over-representation step returns exactly 10 dysregulated pathways, all with enrichment p < 0.05.
- benchmark_differential_table.csv exists, is valid CSV, and contains 35 metabolite rows plus 10 pathway rows with log2FC/enrichment-p and direction/significance columns.
- Every flagged metabolite satisfies both thresholds: p < 0.05 AND |log2FC| > 1.

## Limitations

- Thresholds are workflow-specific (p < 0.05, |log2FC| > 1, enrichment p < 0.05); other studies may use different cutoffs, so the skill must be re-parameterised for new case studies.
- The directionality check depends on the article's named metabolite list — metabolites not in that list cannot be direction-verified against the source.
- The pathway enrichment relies on the pathway database bundled with PlantMetSuite; coverage gaps in that database could miss dysregulated pathways.

## Evidence

- [results] Named-metabolite directions: [evidence span withheld — anchor status: not_found]
- [abstract] PlantMetSuite scope: "PlantMetSuite encompasses interactive bioinformatics tools and databases specifically tailored to plant metabolomics data, facilitating upstream-to-downstream analysis in metabolomics and supporting"
- [abstract] PlantMetSuite accessibility: "PlantMetSuite can be accessed directly through a user's browser without the need for installation or programming skills."
- [abstract] PlantMetSuite tool role: "we developed PlantMetSuite, a web-based tool for comprehensive metabolomics analysis and visualisation"
