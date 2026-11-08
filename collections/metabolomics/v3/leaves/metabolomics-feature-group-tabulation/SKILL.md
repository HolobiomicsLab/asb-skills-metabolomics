---
name: metabolomics-feature-group-tabulation
description: Use when a khipu annotation (JSON empirical-compound trees) of LC-MS
  isotope-tracing data must be parsed into per-compound isotope-index distributions
  for labeled vs unlabeled sample groups.
when_to_use_negative:
- The input is still raw or unannotated LC-MS features — run khipu preannotation first
  (e.g. `khipu -i ecoli_pos.tsv -o out`); this skill operates on empirical-compound
  trees, not on raw feature lists
- Complete downstream annotation is needed (contaminants, authentic libraries, tandem
  MS/MS data); khipu preannotation is explicitly a precursor step, and full annotation
  requires those additional resources
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: khipu
  role: Produces the empirical-compound (tree) annotations — isotope branches grouped
    per adduct and re-aligned to the khipu grid — that this skill parses and tabulates;
    run as `khipu -i ecoli_pos.tsv -o this_test`
  repo: github.com/shuzhao-li-lab/khipu
- name: asari
  role: Upstream preprocessing (v1.9.2) generating the feature table (ecoli_pos.tsv)
    that khipu consumes; also uses khipu internally for preannotation
- name: Python 3
  role: Scripting environment for parsing the JSON empirical-compound trees and building
    the per-compound isotope-index distribution table
- name: Jupyter
  role: Notebook environment for data analysis and enhanced tree/grid visualization
    when tabulating isotope indices
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:a02bbdea9e7246211b329881af7cda45dcacc2e230bd3cdb5998618404b2679d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Jupyter
  - Python 3
  - asari
  - khipu
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolomics-feature-group-tabulation

## Summary

Tabulates khipu-annotated empirical compounds — each a tree of isotope branches and adduct trunk — from LC-MS feature tables (e.g. ecoli_pos.tsv) so isotope indices (13C/12C*1, *2, …) can be extracted per compound across labeled and unlabeled sample groups.

## When to use

Use when a khipu annotation (JSON empirical-compound trees) of LC-MS isotope-tracing data must be parsed into per-compound isotope-index distributions for labeled vs unlabeled sample groups.

## When NOT to use

- The input is still raw or unannotated LC-MS features — run khipu preannotation first (e.g. `khipu -i ecoli_pos.tsv -o out`); this skill operates on empirical-compound trees, not on raw feature lists
- Complete downstream annotation is needed (contaminants, authentic libraries, tandem MS/MS data); khipu preannotation is explicitly a precursor step, and full annotation requires those additional resources

## Inputs

- khipu JSON annotation of empirical compounds (trees) from LC-MS feature table, e.g. ecoli_pos.tsv preannotated with asari 1.9.2
- Feature table (tab-delimited: feature ID, m/z, rtime, intensity columns) used as khipu input, with explicit start/end intensity column indices
- List of sample columns distinguishing unlabeled vs isotope-labeled groups (e.g. three unlabeled and three U-13C-glucose samples)

## Outputs

- Per-compound isotope-index distribution table, e.g. isotope_index_distribution.csv, one row per empirical compound (548 for the E. coli reverse-phase ESI+ set) with isotope-index intensities across samples
- Neutral mass of the M0 compound per khipu, obtained by linear regression over matched m/z values
- Isotopologue envelope checks per compound, e.g. M0 peak for unlabeled and M9 (13C/12C*9) peak for U-13C-labeled compounds with known carbon count

## How to apply

Load the khipu JSON output (e.g. 548 empirical compounds annotated from 1,745 of 3,602 features in the E. coli reverse-phase ESI+ data processed with asari 1.9.2) and parse each empirical-compound tree. Extract the isotope branches (13C/12C, 13C/12C*2, …) attached to each adduct on the trunk, using the 1.0034 Da 13C/12C mass spacing as the isotope-edge criterion; isotope edges form their own groups by shared nodes, each group belonging to one adduct type and treated as one branch. Assign each branch to the adduct trunk and re-align isotopes to the khipu grid (M0, 13C/12C, 13C/12C*2, …) before regressing the neutral mass of the M0 compound. Then tabulate, per compound, the isotope-index intensities across the sample columns (e.g. three unlabeled and three U-13C-glucose samples for the ecoli_pos.tsv data set), writing a table such as isotope_index_distribution.csv. Validate by checking that the table contains 548 compound rows, that the multi-isotope count matches the reported 445 compounds with multiple isotopes, and that the compound with inferred neutral mass 187.1686 (acetylspermidine, C9H21N3O, 9 carbons) peaks at M0 in unlabeled and at 13C/12C*9 in U-13C-labeled samples.

## Related tools

- **khipu** (Produces the empirical-compound (tree) annotations — isotope branches grouped per adduct and re-aligned to the khipu grid — that this skill parses and tabulates; run as `khipu -i ecoli_pos.tsv -o this_test`) — github.com/shuzhao-li-lab/khipu
- **asari** (Upstream preprocessing (v1.9.2) generating the feature table (ecoli_pos.tsv) that khipu consumes; also uses khipu internally for preannotation)
- **Python 3** (Scripting environment for parsing the JSON empirical-compound trees and building the per-compound isotope-index distribution table)
- **Jupyter** (Notebook environment for data analysis and enhanced tree/grid visualization when tabulating isotope indices)

## Examples

```
khipu -i testdata/ecoli_pos.tsv -o this_test
```

## Evaluation signals

- Row count invariant: the isotope_index_distribution.csv contains exactly 548 compound rows for the E. coli reverse-phase ESI+ data set (khipu annotated 548 empirical compounds from 1,745 of 3,602 features)
- Multi-isotope count matches the reported 445 compounds with multiple isotopes out of 548
- Envelope check for compound with inferred neutral mass 187.1686 (acetylspermidine, C9H21N3O): unlabeled samples peak at M0, U-13C-labeled samples peak at 13C/12C*9 (M9), consistent with 9 carbons
- Every isotope branch is assigned to exactly one adduct on the trunk; isotope groups share nodes and each group belongs to a single adduct type
- Isotope-index spacing matches the khipu grid offsets (13C/12C = 1.0034 Da; M+H[+] offset 1.007276; M+Na[+] offset 22.989276) within the configured ppm tolerance

## Limitations

- Annotation of untargeted metabolomics data, including isotopic tracing data, is still not fully solved; khipu preannotation is a key step but not complete annotation
- Complete annotation still requires additional resources beyond khipu: contaminants, authentic libraries, and tandem MS data
- Search patterns for isotopes are experiment-dependent; users must overwrite defaults by supplying their own search patterns (see khipu demo notebooks), otherwise labeled nuclei outside the default 13C grid are missed
- Some ions may enter the initial network by mistake or as unresolved signals; they are removed from the established khipu and sent off to form a new khipu, so a compound's tree may not be fully explained
- Benchmarking is deferred to future work, so performance claims cannot be relied on yet

## Evidence

- [readme] Isotope branch assignment to trunk and grid re-alignment: [evidence span withheld — anchor status: not_found]
