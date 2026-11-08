---
name: taxonomic-profile-parsing
description: Use when input is a CAMI-format taxonomic profile (taxID, rank, abundance
  per taxon) that must be mapped onto the NCBI taxdump tree, e.g. for TAMPA-style
  tree annotation or profiler comparison.
when_to_use_negative:
- Input is amplicon microbial data — metacoder-style tools and this WGS-focused pipeline
  are not interchangeable; TAMPA is designed for WGS metagenomic taxonomic profiles
- Input profiles are not in CAMI standardized format or lack taxIDs, so taxa cannot
  be attached to the NCBI taxdump tree
- Only a single-value metric comparison (e.g. F1 score, UniFrac) is needed without
  per-clade abundance visualization
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3174
tools:
- name: TAMPA
  role: Parses CAMI-format taxonomic profiles and, with the NCBI taxdump database,
    creates percentage abundance predictions for each taxon before tree building
  repo: https://github.com/dkoslicki/TAMPA.git
- name: ete3
  role: Provides the NCBI taxonomy interface used to attach profile taxIDs to tree
    nodes and construct the annotated tree
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:951636f6599c34eaf84c82479296795476b48c82cdab50bd696edce971bce9d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - TAMPA
  - ete3
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# taxonomic-profile-parsing

## Summary

Parses CAMI-format taxonomic profile files (e.g. sample CAMI_HIGH_S001) to extract per-taxon taxonomic rank assignments and percentage abundances, and attaches each taxon to the NCBI taxonomy using the ete3 toolkit's NCBI taxonomy interface so abundances can be placed on taxonomy tree nodes.

## When to use

Use when input is a CAMI-format taxonomic profile (taxID, rank, abundance per taxon) that must be mapped onto the NCBI taxdump tree, e.g. for TAMPA-style tree annotation or profiler comparison.

## When NOT to use

- Input is amplicon microbial data — metacoder-style tools and this WGS-focused pipeline are not interchangeable; TAMPA is designed for WGS metagenomic taxonomic profiles
- Input profiles are not in CAMI standardized format or lack taxIDs, so taxa cannot be attached to the NCBI taxdump tree
- Only a single-value metric comparison (e.g. F1 score, UniFrac) is needed without per-clade abundance visualization

## Inputs

- CAMI-format taxonomic profile file (e.g. .profile.txt with taxID, rank, and percentage abundance per taxon)
- NCBI taxdump database dump file (via -d/--db_file)
- Taxonomic rank to plot at (e.g. class, phylum)
- Sample of interest identifier (e.g. CAMI_HIGH_S001, marmgCAMI2_short_read_sample_0)

## Outputs

- Per-taxon percentage abundance predictions mapped onto NCBI taxonomy tree nodes
- Annotated taxonomy tree structure for downstream TAMPA visualization (plot output such as png/svg/pdf)

## How to apply

Parse the CAMI-format profile file, extracting the taxonomic rank assignments (the rank passed as the positional 'taxonomic_rank' argument, e.g. class or phylum) and the percentage abundance per taxon. Load the NCBI taxdump database (via the -d/--db_file option) and use the ete3 toolkit's NCBI taxonomy interface to attach each taxID in the profile to its corresponding tree node. Compute the percentage abundance predictions per taxon and place them on the corresponding nodes, optionally applying a scaling key such as -k linear and a resolution such as -r 1600 as in the TAMPA run command. The parsed, annotated profile/tree is then ready for TAMPA's normal or contrast visualization modes.

## Related tools

- **TAMPA** (Parses CAMI-format taxonomic profiles and, with the NCBI taxdump database, creates percentage abundance predictions for each taxon before tree building) — https://github.com/dkoslicki/TAMPA.git
- **ete3** (Provides the NCBI taxonomy interface used to attach profile taxIDs to tree nodes and construct the annotated tree)

## Examples

```
conda create -c etetoolkit -y -n CAMIViz python=3.7 numpy ete3 seaborn pandas matplotlib biom-format
```

## Evaluation signals

- Output tree file exists and parses as valid Newick
- Tree contains nodes corresponding to the taxIDs present in the input CAMI profile
- Extracted percentage abundances per taxon match the values in the source profile for the specified sample (e.g. CAMI_HIGH_S001)
- Every taxID in the profile resolves successfully against the NCBI taxdump via ete3 (no unmapped taxa)
- Resulting node abundances are consistent with the chosen scaling key (e.g. -k linear) used in the TAMPA run

## Limitations

- Validation in the paper was limited to simulated data (e.g. CAMI high-complexity sample 1), where the true ground-truth distribution is only known for simulated data
- Profiles from profilers that do not emit standardized CAMI-format output or valid taxIDs cannot be attached to the NCBI taxonomy

## Evidence

- [methods] profile files and taxdump: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [methods] ete3 tree-building: "These relative abundances are then used as input to the tree-building pipeline, which uses the ete3 toolkit to make the desired plot."
- [results] percentage relative abundance per taxa: "TAMPA does this by computing the percentage relative abundance per taxa and identifying which specific clades contributed to metric values"
- [methods] CAMI_HIGH_S001 sample: "We limited our analysis to sample 1 of the high-complexity dataset, denoted by CAMI_HIGH_S001."
- [readme] db_file option: "specify database dump file"
