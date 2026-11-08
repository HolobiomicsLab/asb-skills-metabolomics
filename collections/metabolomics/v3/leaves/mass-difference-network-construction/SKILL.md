---
name: mass-difference-network-construction
description: Use when you have a feature table from untargeted metabolomics or isotope-tracing
  LC-MS (feature ID, m/z, rtime, intensities) and need to group degenerate ions into
  empirical compounds before formula/identity assignment.
when_to_use_negative:
- Data are raw vendor files (mzML/Thermo raw) rather than a preprocessed feature table
  — run a peak-picking tool like asari 1.9.2 first
- A prior chemical formula or assumed base ion is already known for the features —
  direct formula assignment tools are then more appropriate
- Complete, high-confidence metabolite identification requiring authentic libraries
  or tandem MS data is the goal — this skill only performs generalized preannotation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: khipu
  role: Python 3 package implementing the mass-difference search, network construction,
    subnetwork partitioning, and tree conversion for empirical-compound preannotation
  repo: https://github.com/shuzhao-li/khipu
- name: mass2chem
  role: Provides the search functions over the feature list that find pattern-matched
    feature pairs for the mass distance network
  repo: https://github.com/shuzhao-li-lab/mass2chem
- name: networkx
  role: Supports the graph operations used to build and partition the mass distance
    network
- name: asari
  role: Produces the input feature table (v1.9.2) that is fed into the network-construction
    step
  repo: https://github.com/shuzhao-li/asari
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:e72df042dad641ac02e5c27f81b192a890f5d9056758d044e3eee4fce7cb3949
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shuzhao-li/khipu
  tools:
  - Jupyter
  - Python 3
  - asari
  - khipu
  - mass2chem
  - networkx
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mass-difference-network-construction

## Summary

Builds an untargeted mass-distance network over LC-MS feature m/z values by searching isotope mass differences (1.0034 Da for 13C/12C) and adduct mass differences (e.g. 22.9893 Da for Na+) to connect ion pairs as the first step of khipu preannotation.

## When to use

Use when you have a feature table from untargeted metabolomics or isotope-tracing LC-MS (feature ID, m/z, rtime, intensities) and need to group degenerate ions into empirical compounds before formula/identity assignment.

## When NOT to use

- Data are raw vendor files (mzML/Thermo raw) rather than a preprocessed feature table — run a peak-picking tool like asari 1.9.2 first
- A prior chemical formula or assumed base ion is already known for the features — direct formula assignment tools are then more appropriate
- Complete, high-confidence metabolite identification requiring authentic libraries or tandem MS data is the goal — this skill only performs generalized preannotation

## Inputs

- Tab-delimited LC-MS feature table with columns: feature ID, m/z, retention time, then intensity columns (e.g. asari 1.9.2 output like testdata/ecoli_pos.tsv)
- User-defined isotope search patterns (e.g. 1.0034 Da per 13C/12C) and adduct search patterns (e.g. 22.9893 Da for Na+)
- Mass precision in ppm (mz_tolerance_ppm) and retention-time tolerance (--rtol)
- Ionization mode (pos or neg) and start/end column indices of intensity data

## Outputs

- An overall mass distance network partitioned into connected subnetworks, one per adduct-isotope group
- A set of khipu instances, each representing one 'empirical compound' for downstream annotation
- Transparent JSON and tab-delimited output files (e.g. this_test.json and this_test.tsv) listing annotated empirical compounds

## How to apply

Take a tab-delimited feature table (e.g. asari 1.9.2 output like ecoli_pos.tsv with 3,602 features) whose first columns are feature ID, m/z, rtime, followed by intensity columns. Search the m/z list for all pairs matching any user-defined isotope or adduct mass-difference pattern — e.g. 1.0034 Da per 13C/12C step and 22.9893 Da for Na+ — within a mass precision set via the --ppm tolerance. Connect all pattern-matched pairs into one overall network, then partition it into connected subnetworks, exploiting that all ions connected by isotopic edges belong to the same adduct so per-adduct isotopic subnetworks can be defined. The initial adduct patterns are trimmed to reduce ambiguity, and ions entering the network by mistake or as unresolved signals are later removed from an established khipu and sent off to form a new one. Each connected subnetwork then becomes a khipu instance (one 'empirical compound') for downstream tree conversion, isotope/adduct grid alignment, and neutral-mass regression.

## Related tools

- **khipu** (Python 3 package implementing the mass-difference search, network construction, subnetwork partitioning, and tree conversion for empirical-compound preannotation) — https://github.com/shuzhao-li/khipu
- **mass2chem** (Provides the search functions over the feature list that find pattern-matched feature pairs for the mass distance network) — https://github.com/shuzhao-li-lab/mass2chem
- **networkx** (Supports the graph operations used to build and partition the mass distance network)
- **asari** (Produces the input feature table (v1.9.2) that is fed into the network-construction step) — https://github.com/shuzhao-li/asari

## Examples

```
khipu -i testdata/ecoli_pos.tsv -o this_test
```

## Evaluation signals

- For ecoli_pos.tsv (3,602 features, asari 1.9.2), the downstream khipu annotation must yield exactly 548 empirical compounds from 1,745 features with 1,857 singletons and 445 empirical compounds with multiple isotopes
- Yeast ESI+ and ESI- tables should produce 1,775 and 908 empirical compounds respectively under the default adduct list
- Every subnetwork must convert to a directed acyclic graph with a unique root and at most one predecessor per node — edges violating DAG/rooted-tree constraints are weeded out
- Output files must exist in both JSON (khipu transparent empCpd format) and tab-delimited formats with feature ID, m/z, rtime and intensities
- Refit/sanity check: after khipu grid alignment, the neutral M0 mass obtained by linear regression should reconcile adduct shifts like M+H (1.007276) and M+Na (22.989276) to within the --ppm tolerance

## Limitations

- Many current tools that take a network approach depend on assumed base ions or formulas to assign relationships between ions — this skill's value is that khipu does not require prior formula knowledge, but the method is still a preannotation, not a full annotation
- Erroneous edges arise because some ions enter the initial network by mistake or as unresolved signals; these must be removed from the established khipu and re-formed into a new one
- The search pattern for isotopes is experiment-dependent (e.g. which labeling scheme was used), so users must overwrite the default patterns for non-13C tracing or derivatization experiments
- Benchmarking of the method was deferred to future work, so absolute accuracy across datasets is not yet established

## Evidence
