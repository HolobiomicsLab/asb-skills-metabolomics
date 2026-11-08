---
name: tool-inventory-comparison-across-frameworks
description: Use when the analysis goal is to determine which analytical tools overlap
  across two or more Galaxy-based (or standalone) metabolomics toolkits, and which
  tools are unique to one suite.
when_to_use_negative:
- The goal is to execute a metabolomics analysis on actual data (feature tables, spectra)
  rather than to compare which tools exist in each framework
- Only one framework's inventory is available — overlap and uniqueness claims require
  at least two suites to compare
- The input is raw untargeted metabolomics data needing preprocessing (peak picking,
  imputation, normalization) — use the frameworks' own tools instead
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0091
tools:
- name: SECIMTools
  role: Framework whose tool inventory is parsed and compared; source of claimed-unique
    tools (BFF, RT flags, ROR, LASSO, Elastic Net)
  repo: https://github.com/SECIMTools
- name: Galaxy-M
  role: Comparison framework covering peak-picking/feature identification and data
    pre-processing
- name: Workflow4Metabolomics
  role: Comparison framework focused on feature annotation
- name: Galaxy
  role: Shared web platform into which the compared tool suites are wrapped, enabling
    the cross-framework comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:ee2c2b1233421924bcb1d9c209fc23b9045db6cb8a2387ed3646838f95366f24
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - Galaxy-M
  - SECIMTools
  - Workflow4Metabolomics
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tool-inventory-comparison-across-frameworks

## Summary

A method for systematically comparing analytical tool coverage across metabolomics frameworks (e.g., SECIMTools, Galaxy-M, Workflow4Metabolomics) by parsing each suite's tool inventory into a tools-by-framework presence matrix, identifying tools shared by all suites and tools unique to one suite.

## When to use

Use when the analysis goal is to determine which analytical tools overlap across two or more Galaxy-based (or standalone) metabolomics toolkits, and which tools are unique to one suite.

## When NOT to use

- The goal is to execute a metabolomics analysis on actual data (feature tables, spectra) rather than to compare which tools exist in each framework
- Only one framework's inventory is available — overlap and uniqueness claims require at least two suites to compare
- The input is raw untargeted metabolomics data needing preprocessing (peak picking, imputation, normalization) — use the frameworks' own tools instead

## Inputs

- Tool inventories or tool lists extracted from each framework's documentation or article text (SECIMTools, Galaxy-M, Workflow4Metabolomics)
- Article text describing each framework's analytical scope (e.g., peak-picking, feature annotation, statistical analysis)

## Outputs

- tools x frameworks presence matrix (tool_presence_matrix.csv)
- List of tools present in all compared suites (e.g., ANOVA, PCA)
- List of tools unique to a single suite (e.g., SECIMTools' BFF, RT flags, LASSO, Elastic Net)

## How to apply

Parse tool inventories from each framework's documentation or article text (e.g., from the SECIMTools abstract listing QC metrics, visualization, statistical methods, and classification/variable-selection tools; from Galaxy-M's peak-picking/pre-processing scope; from Workflow4Metabolomics' feature-annotation focus). Build a tools x frameworks matrix using string matching on tool names, then add presence-in-all-three flags for candidate shared tools such as single-factor fixed ANOVA and PCA. Verify claimed unique tools (SECIMTools' blank feature filtering, retention time diagnostics, run order evaluation, LASSO, Elastic Net) against the source text before flagging exclusivity. Save the verified matrix as tool_presence_matrix.csv.

## Related tools

- **SECIMTools** (Framework whose tool inventory is parsed and compared; source of claimed-unique tools (BFF, RT flags, ROR, LASSO, Elastic Net)) — https://github.com/SECIMTools
- **Galaxy-M** (Comparison framework covering peak-picking/feature identification and data pre-processing)
- **Workflow4Metabolomics** (Comparison framework focused on feature annotation)
- **Galaxy** (Shared web platform into which the compared tool suites are wrapped, enabling the cross-framework comparison)

## Examples

```
import pandas as pd; df.to_csv("tool_presence_matrix.csv")
```

## Evaluation signals

- tool_presence_matrix.csv contains one row per tool and one column per framework (SECIMTools, Galaxy-M, Workflow4Metabolomics) with binary presence flags
- ANOVA and PCA rows are flagged present in all three frameworks, consistent with the article's overlap finding
- Blank Feature Filtering (BFF), Retention Time (RT) Flags, Run Order Regression (ROR), LASSO, and Elastic Net rows are flagged as SECIMTools-only after verification against source text
- Every string-matched tool entry can be traced back to a verbatim tool mention in the corresponding framework's article text or README
- Uniqueness claims are not asserted for tools whose absence in other frameworks could not be verified from their inventories

## Limitations

- String matching on tool names may miss tools named differently across suites or conflate similarly named tools; uniqueness claims depend on the completeness of each framework's published inventory
- The comparison relies on published article text; the article itself notes missing signals such as unresolved repository references (GitHub ref [27] not resolved to a URL) and absent version numbers for SECIMTools and Galaxy
- Tool inventories evolve over time (the README notes packaging for bioconda and Galaxy), so the matrix reflects the state at publication (2018)

## Evidence

- [other] Single-factor fixed ANOVA and PCA overlap: [evidence span withheld — anchor status: not_found]
- [other] SECIMTools unique QC and variable selection tools: [evidence span withheld — anchor status: not_found]
- [intro] Framework scopes: "Galaxy-M was introduced for peak-picking/feature identification and data pre-processing"
- [abstract] SECIMTools tool inventory: "advanced variable selection tools (least absolute shrinkage and selection operator LASSO and Elastic Net)"
- [methods] BFF tool description: "The Blank Feature Filtering (BFF) tool calculates a limit of detection based upon values for a feature"
- [readme] README tool list: "basic statistical analysis methods (partial least squares - discriminant analysis, analysis of variance)"
