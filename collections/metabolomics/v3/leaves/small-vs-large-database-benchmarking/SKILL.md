---
name: small-vs-large-database-benchmarking
description: Use when you have per-spectrum top-hit annotation results from multiple
  ChemDistiller scorer/filter combinations run against both small and large compound
  databases and need to measure pairwise result overlap.
when_to_use_negative:
- Only a single scorer/filter combination was run, so no pairwise overlap can be computed
- Input is raw tandem MS spectra rather than already-produced annotation result lists
  — run ChemDistiller annotation (annotate.py) first
- The analysis targets overall retrieval-rate accuracy rather than agreement between
  alternative annotation configurations
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ChemDistiller
  role: Produces the per-spectrum top-hit annotation lists (via annotate.py and the
    chemdb.results module) whose overlap is benchmarked across filter and scorer combinations
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: Python
  role: Used with pandas/NumPy/SciPy to load annotated results, compute pairwise top-hit
    set intersections, average per database size, and write result_overlap_table.csv
- name: NumPy
  role: Array computation for per-spectrum top-hit set comparisons and averaging of
    overlap percentages
- name: SciPy
  role: Dependency of the ChemDistiller environment used to run the annotation jobs
    whose outputs are benchmarked
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:a0843f2cab8b59670f041e2ba5a4a9c59b00b5b8f16f2eb563df12a30b717a87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - Python
  - SciPy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# small-vs-large-database-benchmarking

## Summary

Quantifies how much the top-hit annotation lists produced by different ChemDistiller scorer/filter combinations overlap, benchmarked separately against small molecular databases (HMDB, NIST14, MassBank, ChEBI) and large compound databases, to show whether each retrieval method returns distinct subsets of query compounds.

## When to use

Use when you have per-spectrum top-hit annotation results from multiple ChemDistiller scorer/filter combinations run against both small and large compound databases and need to measure pairwise result overlap.

## When NOT to use

- Only a single scorer/filter combination was run, so no pairwise overlap can be computed
- Input is raw tandem MS spectra rather than already-produced annotation result lists — run ChemDistiller annotation (annotate.py) first
- The analysis targets overall retrieval-rate accuracy rather than agreement between alternative annotation configurations

## Inputs

- Per-spectrum top-hit annotation lists from ChemDistiller annotation runs (chemdb.results output; annotated_spectra.json or annotated_spectra folder in ChemDistiller internal hierarchical format)
- Method/filter combination labels: element, formula, and InChI filters crossed with fingerprint, fragprint, and totalscore scorers
- Database-size grouping of runs: small molecular databases (HMDB, NIST14, MassBank, ChEBI) versus large compound databases

## Outputs

- Pairwise overlap matrix of method/filter combinations written to result_overlap_table.csv
- Averaged per-spectrum overlap percentages for the small-database and large-database runs
- Validation flags for pairwise combinations outside the 74–93% (small) and 27–78% (large) reference ranges

## How to apply

Load the per-spectrum top-hit annotation lists produced by upstream ChemDistiller annotation runs (chemdb.results output, e.g. annotated_spectra.json or the internal hierarchical annotated_spectra folder) with pandas. For each spectrum, define the top-hit identifier set for every method/filter combination, i.e. the element, formula, and InChI filters crossed with the fingerprint, fragprint, and totalscore scorers. Compute pairwise overlap percentages between every pair of combinations as the size of the intersection of their top-hit identifier sets relative to the compared lists, then average per-spectrum overlaps separately for the small-database runs (HMDB, NIST14, MassBank, ChEBI) and the large-database runs. Write the pairwise overlap matrix to result_overlap_table.csv and validate the results against the published ranges of 74–93% (small databases) and 27–78% (large databases) from Supplementary Table S5, flagging any pairwise combination that falls outside these ranges. The rationale is that different scorers and candidate filters retrieve partially complementary compound subsets, and the degree of complementarity differs sharply with database size.

## Related tools

- **ChemDistiller** (Produces the per-spectrum top-hit annotation lists (via annotate.py and the chemdb.results module) whose overlap is benchmarked across filter and scorer combinations) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **Python** (Used with pandas/NumPy/SciPy to load annotated results, compute pairwise top-hit set intersections, average per database size, and write result_overlap_table.csv)
- **NumPy** (Array computation for per-spectrum top-hit set comparisons and averaging of overlap percentages)
- **SciPy** (Dependency of the ChemDistiller environment used to run the annotation jobs whose outputs are benchmarked)

## Examples

```
python annotate.py Input_Spectra_folder out --ncpu 5 --delta_mz 20 --max_results 100
```

## Evaluation signals

- Computed pairwise overlap percentages for small molecular databases (HMDB, NIST14, MassBank, ChEBI) fall within the 74–93% range reported in Supplementary Table S5
- Computed pairwise overlap percentages for large databases fall within the 27–78% range reported in Supplementary Table S5
- The overlap matrix is square and symmetric with 100% (self-overlap) on the diagonal
- result_overlap_table.csv contains one row/column per method/filter combination (element, formula, InChI filters × fingerprint, fragprint, totalscore scorers)
- Per-spectrum top-hit identifier sets are consistently defined across all combinations before intersection (same spectrum identifiers)

## Limitations

- Large compound databases yield much wider overlap variation (27–78%) than small databases (74–93%), so benchmark conclusions are database-size dependent
- The max_results limit (default 10) restricts candidates per MS1 peak and may need increasing for large compound databases, which changes the candidate pools being compared
- Spectra must be in ChemDistiller internal hierarchical format; conversion from other formats was not yet implemented at the time of the README
- The overlap statistics are specific to the fingerprint, fragprint, and totalscore scorers and the element, formula, and InChI filters; other configurations are not covered

## Evidence

- [readme] Large-database runtime and candidate volume differ from small databases: [evidence span withheld — anchor status: not_found]
- [abstract] ChemDistiller produces the annotation outputs that are compared: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [abstract] Benchmarked methods achieve state-of-the-art retrieval performance: "show retrievals rates comparable to or exceeding the ones obtainable by the current state-of-the-art solutions in the field while offering higher throughput, scalability and processing speed"
- [readme] Annotation runs are launched per input spectra folder with tunable parameters: [evidence span withheld — anchor status: external_doc]
- [intro] ChemDistiller is the annotation engine generating the per-spectrum results: "__ChemDistiller__ is the high-throughput annotation engine for tandem MS spectra annotation."
