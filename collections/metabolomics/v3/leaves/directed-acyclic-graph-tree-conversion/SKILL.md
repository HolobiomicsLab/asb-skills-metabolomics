---
name: directed-acyclic-graph-tree-conversion
description: Use when you have a mass distance network of LC-MS feature pairs (e.g.
  13C/12C 1.0034 Da isotope steps, 22.9893 Da Na+ adduct shifts) and need to organize
  ions into per-compound trees with inferred neutral masses.
when_to_use_negative:
- When the goal is complete/deep annotation — khipu output is preannotation; complete
  annotation still requires contaminants, authentic libraries, and tandem MS data
- When the input is raw vendor data or LC-MS runs rather than a preprocessed feature
  table (preprocessing with asari 1.9.2 or equivalent must happen first)
- When isotope search patterns of the biochemical experiment differ from defaults
  without supplying custom patterns — users must overwrite defaults via demo-notebook-style
  configuration
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: khipu
  role: Implements the Weavor and Khipu classes containing the main algorithms; networkx
    for graph operations, treelib for tree visualization, mass2chem for search functions
  repo: https://github.com/shuzhao-li/khipu
- name: asari 1.9.2
  role: Preprocessing tool that generates the input feature tables and uses khipu
    for preannotation
  repo: https://github.com/shuzhao-li/asari
- name: Python 3
  role: Language in which the algorithm is implemented as an open-source package installable
    via pip
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:73f147d89dee821fe00e2f00cfe115d11ac58ce2a4d641ba901ae40012416dd0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shuzhao-li/khipu
  tools:
  - Python 3
  - asari
  - asari 1.9.2
  - khipu
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# directed-acyclic-graph-tree-conversion

## Summary

Converts a mass-distance-network subnetwork of pattern-matched LC-MS features into a rooted tree (khipu/empirical compound) via DAG enforcement plus two-step trunk/branch optimization, then infers the neutral M0 mass by linear regression against the khipu grid.

## When to use

Use when you have a mass distance network of LC-MS feature pairs (e.g. 13C/12C 1.0034 Da isotope steps, 22.9893 Da Na+ adduct shifts) and need to organize ions into per-compound trees with inferred neutral masses.

## When NOT to use

- When the goal is complete/deep annotation — khipu output is preannotation; complete annotation still requires contaminants, authentic libraries, and tandem MS data
- When the input is raw vendor data or LC-MS runs rather than a preprocessed feature table (preprocessing with asari 1.9.2 or equivalent must happen first)
- When isotope search patterns of the biochemical experiment differ from defaults without supplying custom patterns — users must overwrite defaults via demo-notebook-style configuration

## Inputs

- Mass distance network of pattern-matched feature pairs (from a feature table such as ecoli_pos.tsv, tab-delimited: feature ID, m/z, rtime, intensity columns)
- Isotope and adduct search patterns (khipu grid, e.g. M+H[+], M+NH4[+], M+Na[+], M+HCl+H[+], M+K[+], M+ACN+H[+] rows with 13C/12C isotope steps)
- Parameters: ionization mode (pos/neg), ppm mass tolerance (--ppm), retention time tolerance (--rtol), intensity start/end columns

## Outputs

- Rooted tree structures (khipus / empirical compounds) with adduct trunk and isotopic branches assigned to grid
- Inferred neutral M0 mass per tree via linear regression on matched m/z values
- Pre-annotation exported as JSON and tab-delimited files (e.g. this_test.json and this_test.tsv)

## How to apply

Partition the mass distance network into per-adduct isotopic subnetworks, exploiting that all ions connected by isotopic edges share the same adduct. Convert each subnetwork to a directed acyclic graph, weeding out edges that violate DAG/rooted-tree constraints (each node has at most one predecessor and there is a unique root). Run the two-step optimization: first establish a 'trunk' of adducts by maximizing the number of nodes explained, then assign isotopic branches and re-align isotopes in all branches to the theoretical khipu grid (e.g. M0 row 1.007276 for M+H[+], 22.989276 for M+Na[+]). Ions that entered the network by mistake are removed from the established khipu and sent off to form a new khipu. Finally, infer the neutral M0 mass by linear regression on the matched m/z values across the adduct/isotope grid, and export empirical compounds as JSON (khipu's data model is metDataModel's 'empirical compound').

## Related tools

- **khipu** (Implements the Weavor and Khipu classes containing the main algorithms; networkx for graph operations, treelib for tree visualization, mass2chem for search functions) — https://github.com/shuzhao-li/khipu
- **asari 1.9.2** (Preprocessing tool that generates the input feature tables and uses khipu for preannotation) — https://github.com/shuzhao-li/asari
- **Python 3** (Language in which the algorithm is implemented as an open-source package installable via pip)

## Examples

```
python3 -m khipu.main -i testdata/ecoli_pos.tsv -o this_test
```

## Evaluation signals

- On ecoli_pos.tsv (3,602 asari 1.9.2 LC-MS features), the independently reconstructed annotation should yield 548 empirical compounds (trees) from 1,745 features, matching khipu's reference output
- DAG check: every node has no more than one predecessor and each tree has a unique root; edges violating these constraints are weeded out
- Neutral mass regression: inferred M0 mass should be consistent across adduct columns of the matched khipu grid (e.g. the U-13C E. coli compound inferred at 187.1686 matching acetylspermidine C9H21N3O)
- Misassigned ions should be removed from established khipus and redirected to form new khipus rather than silently dropped
- Run `python3 -m khipu.test` which downloads and uses ecoli_pos.tsv as the acceptance test

## Limitations

- Annotation of untargeted metabolomics data, including isotopic tracing data, is still not fully solved; benchmarking was deferred to future work
- Some ions enter the initial network by mistake or as unresolved signals and must be removed and re-clustered
- Different applications (isotope tracing, chemical labeling, derivatization) require different parameters and custom isotope search patterns

## Evidence

- [methods] the networks have to become directed acyclic graph (DAG). During this process, erroneous edges are weeded out because they do not satisfy DAG and a rooted tree.: "the networks have to become directed acyclic graph (DAG). During this process, erroneous edges are weeded out because they do not satisfy DAG and a rooted tree."
