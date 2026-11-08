---
name: adduct-isotope-two-tier-tree-alignment
description: 'Use when a feature table from LC-MS untargeted metabolomics or isotope
  tracing must be organized into empirical compounds: pattern-matched feature pairs
  (e.g. 1.0034 Da 13C/12C, 22.9893 Da Na+) form a mass distance network that needs
  tree conversion and neutral mass inference.'
when_to_use_negative:
- When complete metabolite annotation (levels beyond preannotation) is required —
  khipu preannotation must be followed by contaminants, authentic libraries, and tandem
  MS data
- When the input is already an annotated empirical compound list rather than a raw
  feature table from a preprocessing tool
- When isotopic search patterns of the biochemical experiment deviate from defaults
  and the user cannot supply custom patterns (though custom patterns are supported
  via demo notebooks)
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: khipu
  role: Python 3 package implementing the Weavor and Khipu classes that perform mass-distance
    network construction, DAG conversion, trunk/branch tree optimization, grid realignment,
    and neutral mass regression; usable as CLI or library
  repo: https://github.com/shuzhao-li/khipu
- name: asari 1.9.2
  role: Preprocessing tool that generates the feature tables (e.g. ecoli_pos.tsv,
    yeast datasets) used as khipu input; asari itself uses khipu for preannotation
  repo: https://github.com/shuzhao-li/asari
- name: mass2chem
  role: Provides the search functions used by khipu to match mass difference patterns
  repo: https://github.com/shuzhao-li-lab/mass2chem
- name: networkx
  role: Supports the graph operations during network partitioning and DAG/tree conversion
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:0c2a40545891a24039fc4bdb59f825644263b70bf0a2066957cca123326b5bf5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shuzhao-li/khipu
  tools:
  - Python 3
  - asari
  - asari 1.9.2
  - khipu
  - mass2chem
  - networkx
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# adduct-isotope-two-tier-tree-alignment

## Summary

The khipu algorithm converts a mass-distance network of pattern-matched LC-MS features into per-adduct subnetworks, then into a rooted two-tier tree via trunk (adduct) and branch (isotope) optimization, and infers the M0 neutral mass by linear regression against the theoretical khipu grid.

## When to use

Use when a feature table from LC-MS untargeted metabolomics or isotope tracing must be organized into empirical compounds: pattern-matched feature pairs (e.g. 1.0034 Da 13C/12C, 22.9893 Da Na+) form a mass distance network that needs tree conversion and neutral mass inference.

## When NOT to use

- When complete metabolite annotation (levels beyond preannotation) is required — khipu preannotation must be followed by contaminants, authentic libraries, and tandem MS data
- When the input is already an annotated empirical compound list rather than a raw feature table from a preprocessing tool
- When isotopic search patterns of the biochemical experiment deviate from defaults and the user cannot supply custom patterns (though custom patterns are supported via demo notebooks)

## Inputs

- Feature table (tab-delimited TSV) with columns: feature ID, m/z, rtime, followed by intensity columns, e.g. ecoli_pos.tsv preprocessed by asari 1.9.2
- Initial isotope patterns and adduct patterns (khipu grid), e.g. 13C/12C steps and M+H, M+NH4, M+Na, M+HCl+H, M+K, M+ACN+H adducts
- Parameters: ionization mode (pos/neg), ppm mass tolerance, retention time tolerance, start/end intensity columns

## Outputs

- Empirical compounds (khipu trees) in JSON format, e.g. this_test.json
- Empirical compounds in tab-delimited format, e.g. this_test.tsv
- Per-tree inferred M0 neutral masses (e.g. 187.1686 matching acetylspermidine C9H21N3O)

## How to apply

First search the feature list (e.g. an asari 1.9.2 TSV such as ecoli_pos.tsv with 3,602 features) for pairs matching isotope patterns (1.0034 Da steps for 13C/12C) and adduct patterns (e.g. 22.9893 Da for Na+), connecting them into a mass distance network. Partition the network into connected subnetworks, exploiting that all ions connected by isotopic edges belong to the same adduct, so per-adduct subnetworks can be defined. Convert each subnetwork to a directed acyclic graph, weeding out edges that violate DAG/rooted-tree constraints (each node with no more than one predecessor and a unique root). Then run the two-step optimization: build the adduct 'trunk' by aligning the abstracted adduct network to the adduct tree to maximize the number of nodes explained, assign each isotopic group as a 'branch' to the trunk, and re-align isotopes in all branches to establish the optimal match to the khipu grid. Finally, infer the neutral M0 mass of each tree by linear regression on the matched m/z values across the available ions and the theoretical grid, and export the empirical compounds as JSON and tab-delimited files.

## Related tools

- **khipu** (Python 3 package implementing the Weavor and Khipu classes that perform mass-distance network construction, DAG conversion, trunk/branch tree optimization, grid realignment, and neutral mass regression; usable as CLI or library) — https://github.com/shuzhao-li/khipu
- **asari 1.9.2** (Preprocessing tool that generates the feature tables (e.g. ecoli_pos.tsv, yeast datasets) used as khipu input; asari itself uses khipu for preannotation) — https://github.com/shuzhao-li/asari
- **mass2chem** (Provides the search functions used by khipu to match mass difference patterns) — https://github.com/shuzhao-li-lab/mass2chem
- **networkx** (Supports the graph operations during network partitioning and DAG/tree conversion)

## Examples

```
khipu -i testdata/ecoli_pos.tsv -o this_test
```

## Evaluation signals

- Reconstructed tree count matches khipu reference output: 548 empirical compounds from 1,745 of 3,602 features on ecoli_pos.tsv
- Every tree satisfies rooted-tree constraints: each node has no more than one predecessor and there is a unique root after erroneous edges are weeded out
- Inferred neutral masses are chemically plausible, e.g. M0 187.1686 matching acetylspermidine (C9H21N3O, 9 carbons) in the U-13C-labeled E. coli data
- Output files this_test.json and this_test.tsv are produced with the empirical compound data model from metDataModel
- For yeast datasets, annotation counts and explained features fall in reported ranges (1,775 ESI+ / 908 ESI- empirical compounds; 8,049 and 2,912 explained features with additional adducts)

## Limitations

- Some ions may enter the initial network by mistakes or unresolved signals; they are removed from the established khipu and sent off to form a new khipu
- Benchmarking is deferred to future work; annotation of untargeted metabolomics including isotopic tracing data is still not fully solved
- Different applications (regular LC-MS vs isotope tracing vs chemical derivatization) require different parameters and search patterns

## Evidence

- [abstract] An algorithm is presented to convert mass distance networks to this tree structure with high fidelity.: "An algorithm is presented to convert mass distance networks to this tree structure with high fidelity."
- [methods] the networks have to become directed acyclic graph (DAG). During this process, erroneous edges are weeded out: "the networks have to become directed acyclic graph (DAG). During this process, erroneous edges are weeded out because they do not satisfy DAG and a rooted tree."
- [methods] Based on the matched m/z values, the neutral mass of the M0 compound is obtained by a regression model.: "Based on the matched m/z values, the neutral mass of the M0 compound is obtained by a regression model."
- [readme] Establish a "trunk" of adducts with a root and a path for adducts, by optimizing the number of nodes explained.: "Establish a "trunk" of adducts with a root and a path for adducts, by optimizing the number of nodes explained."
- [readme] Some ions may come into the initial network by mistakes or unresolved signals.: "Some ions may come into the initial network by mistakes or unresolved signals.
The are removed from the established khipu, and sent off to form a new khipu."
