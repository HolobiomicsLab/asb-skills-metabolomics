---
name: isotopic-subnetwork-partitioning
description: Use when you have a preprocessed LC-MS feature table (e.g. asari 1.9.2
  TSV with feature ID, m/z, rtime, intensities) and need to group adduct/isotope-related
  features into empirical compounds with inferred neutral M0 masses, without prior
  formulas.
when_to_use_negative:
- Input is raw LC-MS data or centroided spectra rather than a preprocessed feature
  table — run a preprocessing tool such as asari 1.9.2 first
- 'The task requires complete metabolite identification: khipu provides preannotation
  only, and complete annotation still requires contaminants, authentic libraries,
  and tandem MS data'
- The experiment uses isotope/adduct patterns absent from the default khipu grid without
  supplying custom search patterns (e.g. specialized tracing designs)
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: khipu
  role: Implements the Weavor and Khipu classes containing the subnetwork-to-tree
    algorithms; runnable as CLI or library
  repo: https://github.com/shuzhao-li/khipu
- name: mass2chem
  role: Provides the search functions used to find pattern-matched feature pairs by
    mass difference
  repo: https://github.com/shuzhao-li-lab/mass2chem
- name: networkx
  role: Supports the graph operations for building, partitioning, and pruning the
    mass distance network into DAGs
- name: asari
  role: Produces the input feature tables (v1.9.2 used for all data sets); uses khipu
    internally for preannotation
  repo: https://github.com/shuzhao-li/asari
- name: metDataModel
  role: Defines the 'empirical compound' data model used for the JSON output
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:a756d8553611c64b3d8834e87b1025fade1e859892555e4a412f54b0f2db7c75
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shuzhao-li/khipu
  tools:
  - Python 3
  - asari
  - khipu
  - mass2chem
  - metDataModel
  - networkx
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# isotopic-subnetwork-partitioning

## Summary

Partitions a mass distance network of LC-MS feature pairs into per-adduct isotopic subnetworks and converts each into a rooted tree (khipu) to organize degenerate ions around a compound and infer its neutral mass via linear regression on the khipu grid.

## When to use

Use when you have a preprocessed LC-MS feature table (e.g. asari 1.9.2 TSV with feature ID, m/z, rtime, intensities) and need to group adduct/isotope-related features into empirical compounds with inferred neutral M0 masses, without prior formulas.

## When NOT to use

- Input is raw LC-MS data or centroided spectra rather than a preprocessed feature table — run a preprocessing tool such as asari 1.9.2 first
- The task requires complete metabolite identification: khipu provides preannotation only, and complete annotation still requires contaminants, authentic libraries, and tandem MS data
- The experiment uses isotope/adduct patterns absent from the default khipu grid without supplying custom search patterns (e.g. specialized tracing designs)

## Inputs

- Tab-delimited feature table (feature ID, m/z, rtime, intensity columns) from asari 1.9.2 or any preprocessing tool, e.g. ecoli_pos.tsv (3,602 LC-MS features)
- Initial lists of isotope patterns and adduct patterns defining the khipu grid (defaults built in; custom patterns can be supplied for isotope tracing)

## Outputs

- Empirical compounds (khipu trees) exported as JSON (transparent 'empirical compound' data model of metDataModel) and tab-delimited files, e.g. this_test.json and this_test.tsv
- Neutral M0 mass per tree obtained by linear regression on the khipu grid
- Plain-text tree/grid visualizations for inspection

## How to apply

First search the feature list for mass difference patterns (1.0034 Da for 13C/12C isotope steps, 22.9893 Da for Na+ adducts, using mass2chem search functions within user-set ppm tolerance) and connect matched pairs into an overall network, then partition it into connected subnetworks. Exploit the invariant that all ions connected by isotopic edges belong to the same adduct to define per-adduct isotopic subnetworks. Convert each subnetwork into a directed acyclic graph, weeding out edges that violate the DAG/rooted-tree constraints (each node with no more than one predecessor, a unique root). Run the two-step optimization: establish the adduct 'trunk' with a root by maximizing the number of nodes explained, assign each isotopic group as a 'branch' to the trunk, then re-align isotopes in all branches to the theoretical khipu grid (e.g. M+H[+], M+NH4[+], M+Na[+], M+HCl+H[+], M+K[+], M+ACN+H[+] with up to six 13C/12C steps). Finally, infer the neutral M0 mass by linear regression on the matched m/z values against the khipu grid; erroneous ions that entered the network by mistake are removed and may seed new khipu instances.

## Related tools

- **khipu** (Implements the Weavor and Khipu classes containing the subnetwork-to-tree algorithms; runnable as CLI or library) — https://github.com/shuzhao-li/khipu
- **mass2chem** (Provides the search functions used to find pattern-matched feature pairs by mass difference) — https://github.com/shuzhao-li-lab/mass2chem
- **networkx** (Supports the graph operations for building, partitioning, and pruning the mass distance network into DAGs)
- **asari** (Produces the input feature tables (v1.9.2 used for all data sets); uses khipu internally for preannotation) — https://github.com/shuzhao-li/asari
- **metDataModel** (Defines the 'empirical compound' data model used for the JSON output) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
khipu -i testdata/ecoli_pos.tsv -o this_test
```

## Evaluation signals

- On ecoli_pos.tsv (asari 1.9.2, 3,602 features), the independently implemented annotation must yield 548 empirical compounds (trees) from 1,745 features, matching khipu's reference output
- Every converted subnetwork must satisfy the rooted-tree constraints: each node has at most one predecessor and there is a unique root
- Each isotopic group (branch) must attach to a single adduct trunk, consistent with the invariant that ions connected by isotopic edges share one adduct
- Inferred neutral M0 masses from linear regression should be self-consistent across the ions mapped to the khipu grid (e.g. U-13C acetylspermidine tree inferred at 187.1686 matching C9H21N3O)
- Yeast cross-check: annotation should give 1,775 (ESI+) and 908 (ESI-) empirical compounds; including additional adducts raises explained ESI+ features from 6,310 to 8,049

## Limitations

- Some ions enter the initial network by mistake or represent unresolved signals; they must be removed from the established khipu and may form new khipu instances
- Initial adduct patterns must be trimmed to reduce ambiguity; the core construction should be completed before extending the grid with additional ions
- Benchmarking was deferred to future work, and annotation of untargeted metabolomics data including isotopic tracing is stated as still not fully solved
- Isotope search patterns are experiment-dependent; users must overwrite defaults for specific tracing designs (custom parameters via --ppm, --rtol, --mode)

## Evidence
