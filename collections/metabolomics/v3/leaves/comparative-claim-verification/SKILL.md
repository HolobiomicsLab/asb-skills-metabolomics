---
name: comparative-claim-verification
description: Decision support for claims like 'ANOVA and PCA are in all three Galaxy
  metabolomics toolkits' or 'SECIMTools uniquely offers LASSO and Elastic Net' — trigger
  when a finding asserts presence/absence of a tool across ≥2 named software suites.
when_to_use_negative:
- The claim concerns within-suite tool functionality or parameter defaults rather
  than cross-framework presence/absence
- Only one framework's inventory is available, so 'present in all three' or 'unique
  to' assertions cannot be checked
- The claim is a quantitative performance comparison (runtime, accuracy) rather than
  a tool-coverage statement
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0091
tools:
- name: SECIMTools
  role: Primary suite whose inventory (QC tools such as BFF and RT Flags, statistics
    such as ANOVA and PLS-DA, and variable-selection tools LASSO and Elastic Net)
    is compared against Galaxy-M and Workflow4Metabolomics
  repo: https://github.com/SECIMTools
- name: Galaxy-M
  role: Comparison framework whose peak-picking/pre-processing inventory is parsed
    to test shared-tool and uniqueness claims
- name: Workflow4Metabolomics
  role: Comparison framework whose feature-annotation inventory is parsed to test
    shared-tool and uniqueness claims
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2134-1
    title: 'SECIMTools: a suite of metabolomics data analysis tools'
  source_content_hash: sha256:23cb6d2705cf870c2a4db3e0e9ed28fa60a665172a5be1f789a93537251b6058
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy-M
  - SECIMTools
  - Workflow4Metabolomics
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# comparative-claim-verification

## Summary

Verifies cross-paper comparative claims (e.g., 'tool X appears in all three suites', 'tool Y is unique to SECIMTools') by building a tool-by-framework presence matrix from parsed tool inventories and checking each claim against it, as done when comparing SECIMTools with Galaxy-M and Workflow4Metabolomics.

## When to use

Decision support for claims like 'ANOVA and PCA are in all three Galaxy metabolomics toolkits' or 'SECIMTools uniquely offers LASSO and Elastic Net' — trigger when a finding asserts presence/absence of a tool across ≥2 named software suites.

## When NOT to use

- The claim concerns within-suite tool functionality or parameter defaults rather than cross-framework presence/absence
- Only one framework's inventory is available, so 'present in all three' or 'unique to' assertions cannot be checked
- The claim is a quantitative performance comparison (runtime, accuracy) rather than a tool-coverage statement

## Inputs

- Parsed tool inventory lists per framework (from article text, abstracts, and documentation)
- Comparative claims/findings to verify (e.g., presence-in-all-three and unique-to-suite assertions)
- Repository documentation (e.g., SECIMTools README listing included QC, statistical, and variable-selection tools)

## Outputs

- Tools × frameworks presence matrix (tool_presence_matrix.csv) with flags for tools in all three suites
- Verdict per comparative claim: confirmed, refuted, or unresolved
- List of suite-unique tools (e.g., SECIMTools QC filters and LASSO/Elastic Net) verified against competitor inventories

## How to apply

Parse the tool inventories of each framework (e.g., SECIMTools, Galaxy-M, Workflow4Metabolomics) from article text and documentation, then build a tools × frameworks matrix using string matching on normalized tool names. Set explicit presence-in-all-three flags for the claimed shared tools (single-factor fixed ANOVA, PCA) rather than inferring from partial overlaps. Separately verify uniqueness claims: for SECIMTools-specific items such as blank feature filtering (BFF), retention time (RT) flags, run order evaluation, LASSO, and Elastic Net, confirm absence from the comparison inventories before endorsing 'not present in the other two'. Save the verified matrix (e.g., tool_presence_matrix.csv) as auditable evidence, and record any claim that cannot be confirmed from available text as unresolved rather than accepted.

## Related tools

- **SECIMTools** (Primary suite whose inventory (QC tools such as BFF and RT Flags, statistics such as ANOVA and PLS-DA, and variable-selection tools LASSO and Elastic Net) is compared against Galaxy-M and Workflow4Metabolomics) — https://github.com/SECIMTools
- **Galaxy-M** (Comparison framework whose peak-picking/pre-processing inventory is parsed to test shared-tool and uniqueness claims)
- **Workflow4Metabolomics** (Comparison framework whose feature-annotation inventory is parsed to test shared-tool and uniqueness claims)

## Evaluation signals

- tool_presence_matrix.csv contains one row per claimed tool with explicit presence flags for each of the three frameworks, not free-text notes
- Shared-tool claims (single-factor fixed ANOVA, PCA) resolve to presence flags set in all three frameworks
- Every 'unique to SECIMTools' claim (BFF, RT flags, run order evaluation, LASSO, Elastic Net) is backed by explicit absence evidence from both Galaxy-M and Workflow4Metabolomics inventories
- Claims that cannot be matched by string matching against the parsed inventories are recorded as unresolved, not silently marked confirmed
- Tool-name matches survive normalization (case, punctuation, acronym expansion) — spot-check at least BFF vs 'Blank Feature Filtering' and RT flags vs 'Retention Time (RT) tool'

## Limitations

- String matching on tool names can miss matches or create false equivalences when frameworks use different terminology for the same analytical method
- Presence/absence is only as complete as the article text parsed — a tool absent from the cited text may still exist in a framework
- Version numbers and repository identifiers may be missing from the compared texts (e.g., the SECIMTools discussion does not resolve reference [27] to a URL and gives no version), so comparisons cannot be pinned to specific releases

## Evidence

- [intro] Single-factor fixed ANOVA and PCA shared across suites; SECIMTools-specific tools: "New functionality includes: blank feature filtering"
- [abstract] SECIMTools includes variable selection tools (LASSO and Elastic Net): "advanced variable selection tools (least absolute shrinkage and selection operator LASSO and Elastic Net)"
- [methods] Retention Time (RT) tool criteria used when verifying RT-flag uniqueness claims: "The Retention Time (RT) tool uses two criteria: the tool identifies features with the largest coefficients of variation by percentile using a threshold (10% by default)"
- [readme] README inventory confirming SECIMTools suite contents used for matrix population: [evidence span withheld — anchor status: not_found]
- [intro] Comparative positioning of Galaxy-M and Workflow4Metabolomics against SECIMTools: "Galaxy-M was introduced for peak-picking/feature identification and data pre-processing"
