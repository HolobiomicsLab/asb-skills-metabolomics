---
name: chisquare-significance-testing
description: Use when you have a finite set of characterized substrate/product structures
  and a reference database (e.g., LOTUS), and you must report how many structures
  are present plus the statistical significance of that overlap.
when_to_use_negative:
- The goal is de novo pathway reconstruction or mass-feature annotation (e.g., MR-based
  correlation, ClusterONE clustering, or adduct-based LOTUS matching at 20 ppm tolerance)
  rather than validating database overlap of known structures.
- The comparison data are continuous (e.g., Pearson correlations or mutual ranks needing
  exponential-decay edge weighting) rather than countable matched/unmatched categories
  suitable for a chi-square contingency.
- The reference database differs from the fixed LOTUS snapshot (10/10/2023) and you
  need a version-agnostic or MS/MS-spectral-library-based annotation assessment instead.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3311
tools:
- name: MEANtools
  role: Provides the LOTUS SQLite database and the structure-matching logic used to
    determine presence/absence of each of the 374 characterized-reaction structures
  repo: https://github.com/IntegrativeBioinformaticsLab/meantools
- name: LOTUS
  role: Reference metabolite-structure database queried to establish which of the
    374 structures are matched or unmatched
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:1a1052d7d3f1b558effd2b1b5af170bd79dfbb6b2cc99d375ed70835cac90c20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IntegrativeBioinformaticsLab/meantools
  tools:
  - LOTUS
  - MEANtools
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3311
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# chisquare-significance-testing

## Summary

A statistical validation skill that tests whether the observed overlap between a set of characterized biosynthetic reaction structures and a reference metabolite database (LOTUS) is significantly greater than expected, using a chi-square test on the matched/unmatched presence-absence contingency (DF=1).

## When to use

Use when you have a finite set of characterized substrate/product structures and a reference database (e.g., LOTUS), and you must report how many structures are present plus the statistical significance of that overlap.

## When NOT to use

- The goal is de novo pathway reconstruction or mass-feature annotation (e.g., MR-based correlation, ClusterONE clustering, or adduct-based LOTUS matching at 20 ppm tolerance) rather than validating database overlap of known structures.
- The comparison data are continuous (e.g., Pearson correlations or mutual ranks needing exponential-decay edge weighting) rather than countable matched/unmatched categories suitable for a chi-square contingency.
- The reference database differs from the fixed LOTUS snapshot (10/10/2023) and you need a version-agnostic or MS/MS-spectral-library-based annotation assessment instead.

## Inputs

- Combined_small_middle_big_datasets.csv — characterized biosynthetic reaction dataset with 374 substrate/product structures (Zenodo record 15697913)
- LOTUS database in SQLite format (downloaded 10/10/2023) as shipped in the MEANtools GitHub repository

## Outputs

- structure_presence_chisquare.csv — per-structure matched/unmatched table (374 rows) plus the chi-square statistic (32.353), degrees of freedom (1), and p-value (<0.001)

## How to apply

Load the 374 substrate/product structures from the characterized-reaction dataset (Combined_small_middle_big_datasets.csv, Zenodo record 15697913) and the LOTUS database in SQLite format distributed with MEANtools (downloaded 10/10/2023). Match each structure against the LOTUS entries to label it matched or unmatched per structure ID. Tabulate the matched/unmatched counts, confirming the reported 132 of 374 structures present. Compute the chi-square statistic on the 1-degree-of-freedom presence/absence contingency and verify it reproduces 32.353 with p < 0.001. Write the per-structure match table and significance result to structure_presence_chisquare.csv.

## Related tools

- **MEANtools** (Provides the LOTUS SQLite database and the structure-matching logic used to determine presence/absence of each of the 374 characterized-reaction structures) — https://github.com/IntegrativeBioinformaticsLab/meantools
- **LOTUS** (Reference metabolite-structure database queried to establish which of the 374 structures are matched or unmatched)

## Evaluation signals

- The per-structure match table contains exactly 374 rows, one per characterized substrate/product structure ID.
- The matched/unmatched tabulation reproduces 132 structures present out of 374.
- The chi-square statistic equals 32.353 with DF=1.
- The reported p-value is < 0.001, confirming statistical significance of the overlap.
- The output file structure_presence_chisquare.csv contains both the per-structure match labels and the significance result.

## Limitations

- The overlap count and significance are tied to the specific LOTUS snapshot (downloaded 10/10/2023) converted to SQLite for MEANtools; a different database version changes matched/unmatched counts.
- The test treats the 374 structures as a binary matched/unmatched contingency (DF=1) and does not capture finer distinctions such as stereoisomers, which mass spectrometric data cannot resolve.
- The statistic validates database overlap only; it does not confirm that the matched structures support individual predicted reaction steps in a reconstructed pathway.

## Evidence

- [other] 132 of 374 structures were found in LOTUS with statistical significance (chi-square: 32.353, DF=1, p < 0.001).: [evidence span withheld — anchor status: not_found]
- [discussion] The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository.: "The LOTUS database was converted to an SQLite format to be compatible with MEANtools, and it was made available in the GitHub repository."
- [supplementary] can be found at https://zenodo.org/records/15697913/files/Combined_small_middle_big_datasets.csv: "can be found at https://zenodo.org/records/15697913/files/Combined_small_middle_big_datasets.csv"
