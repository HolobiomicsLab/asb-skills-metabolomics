---
name: metabolomics-workflow-tool-mapping
description: 'Use when choosing or comparing metabolomics analysis toolkits: the input
  is textual tool inventories or feature lists from framework documentation/articles,
  and the goal is a structured overlap/unique-tool comparison rather than running
  the tools.'
when_to_use_negative:
- The goal is to execute metabolomics QC or statistics (e.g., running BFF, ANOVA,
  or PCA on a feature table) rather than to map which frameworks offer which tools
- The input is raw metabolomics data requiring peak-picking or pre-processing — this
  skill operates on documented tool inventories, not spectra
- One or more frameworks lacks a documented tool inventory, making string-matched
  presence/absence calls unverifiable
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: SECIMTools
  role: Primary toolkit whose inventory is parsed and compared; source of QC (BFF,
    RT Flags, ROR), statistical (ANOVA, PLS-DA, t-test, Kruskal-Wallis), classification
    (random forest, SVM), and variable selection (LASSO, Elastic Net) tools
  repo: https://github.com/SECIMTools
- name: Galaxy-M
  role: Comparison framework whose documented tool inventory is parsed into the presence
    matrix
- name: Workflow4Metabolomics
  role: Comparison framework whose documented tool inventory is parsed into the presence
    matrix
- name: Galaxy
  role: Shared wrapping platform that makes the compared tools interoperable across
    frameworks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:baeaed2311a5e48386bbdc63a9c81a30449a18e867d11fddec14dd9c71fda64a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Galaxy-M
  - SECIMTools
  - Workflow4Metabolomics
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolomics-workflow-tool-mapping

## Summary

Maps the analytical tool coverage of Galaxy-based metabolomics suites (SECIMTools, Galaxy-M, Workflow4Metabolomics) into a tools × frameworks presence matrix, identifying tools shared across all suites and tools unique to one suite such as SECIMTools' QC and variable selection tools.

## When to use

Use when choosing or comparing metabolomics analysis toolkits: the input is textual tool inventories or feature lists from framework documentation/articles, and the goal is a structured overlap/unique-tool comparison rather than running the tools.

## When NOT to use

- The goal is to execute metabolomics QC or statistics (e.g., running BFF, ANOVA, or PCA on a feature table) rather than to map which frameworks offer which tools
- The input is raw metabolomics data requiring peak-picking or pre-processing — this skill operates on documented tool inventories, not spectra
- One or more frameworks lacks a documented tool inventory, making string-matched presence/absence calls unverifiable

## Inputs

- Tool inventories parsed from framework documentation or article text (SECIMTools, Galaxy-M, Workflow4Metabolomics)
- Framework descriptions stating each toolkit's analytical scope (e.g., peak-picking, pre-processing, feature annotation)

## Outputs

- tool_presence_matrix.csv — a tools × frameworks boolean/presence matrix
- List of tools present in all three suites (e.g., single-factor fixed ANOVA, PCA)
- List of SECIMTools-unique tools (blank feature filtering, RT diagnostics, run order evaluation, LASSO, Elastic Net)

## How to apply

Parse the tool inventories from each framework's article text or documentation (e.g., SECIMTools' QC tools like Blank Feature Filtering and RT Flags, statistics like PLS-DA, ANOVA, t-test, Kruskal-Wallis, classification methods like random forest and SVMs, and variable selection via LASSO and Elastic Net). Build a tools × frameworks matrix using string matching on tool names, normalizing synonyms (e.g., 'principal component analysis' vs 'PCA'). Flag cells for presence in all three frameworks — in the SECIMTools comparison, single-factor fixed ANOVA and PCA appear in all three suites. Verify claims of uniqueness (BFF, RT flags, run order evaluation, LASSO, Elastic Net are SECIMTools-specific) by checking that no counterpart tool appears in the other inventories, then save the result as tool_presence_matrix.csv.

## Related tools

- **SECIMTools** (Primary toolkit whose inventory is parsed and compared; source of QC (BFF, RT Flags, ROR), statistical (ANOVA, PLS-DA, t-test, Kruskal-Wallis), classification (random forest, SVM), and variable selection (LASSO, Elastic Net) tools) — https://github.com/SECIMTools
- **Galaxy-M** (Comparison framework whose documented tool inventory is parsed into the presence matrix)
- **Workflow4Metabolomics** (Comparison framework whose documented tool inventory is parsed into the presence matrix)
- **Galaxy** (Shared wrapping platform that makes the compared tools interoperable across frameworks)

## Examples

```
python -c "import pandas as pd; m = pd.read_csv('tool_inventory_parsed.csv', index_col=0); m.to_csv('tool_presence_matrix.csv')"
```

## Evaluation signals

- Every tool × framework cell in tool_presence_matrix.csv is populated (present/absent), with no unmatched tool names left from string matching
- The all-three-suite intersection contains exactly the expected shared tools (single-factor fixed ANOVA, PCA) and no SECIMTools-only tools
- SECIMTools-unique claims for BFF, RT Flags, run order evaluation, LASSO, and Elastic Net are each backed by absence in both Galaxy-M and Workflow4Metabolomics inventories
- Re-running the string-matching step on the same source text reproduces the same matrix
- Output file is valid CSV with columns for each of the three frameworks

## Limitations

- Tool inventories are derived from article text, so tools not mentioned in the text will be missed as absent
- No version number for SECIMTools or Galaxy is given in the discussion; the GitHub reference [27] is not resolved to a repository URL in the text
- String matching may miss equivalent tools named differently across frameworks (e.g., paraphrased PCA or ANOVA implementations), biasing the unique-tool list

## Evidence

- [intro] Galaxy-M scope: "Galaxy-M was introduced for peak-picking/feature identification and data pre-processing"
- [intro] W4M scope: "Workflow4Metabolomics is a framework that focuses on feature annotation"
- [abstract] SECIMTools as Galaxy-wrapped Python suite: "SECIMTools (SouthEast Center for Integrated Metabolomics) is a set of Python applications that are available both as standalone tools and wrapped for use in Galaxy."
- [readme] SECIMTools tool suite inventory: "The suite includes a comprehensive set of quality
control metrics (retention time window evaluation and various peak evaluation
tools)"
- [methods] Shared Galaxy importability: "The files available in both public repositories can be imported into Galaxy and used in SECIMTools."
