---
name: ncbi-taxonomy-resolution
description: Use when a CAMI-format taxonomic profile contains taxIDs that must be
  attached to an NCBI taxonomy tree (e.g., to annotate nodes with percentage abundances
  or emit a Newick tree for downstream plotting).
when_to_use_negative:
- Input is an amplicon dataset — metacoder-style tools, not TAMPA's NCBI taxdump resolution,
  are the article's stated wrong fit, and TAMPA is not designed for amplicon data
- You only need a single-value benchmarking metric (e.g., F1 score or UniFrac) — taxonomy-tree
  construction is unnecessary
- No NCBI taxdump database is available and none can be supplied via -d, since taxa
  cannot be attached without it
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3174
tools:
- name: TAMPA
  role: Wraps the pipeline that attaches profile taxa to the NCBI taxonomy tree and
    computes per-taxon percentage abundances
  repo: https://github.com/dkoslicki/TAMPA.git
- name: ete3
  role: Toolkit used to load the NCBI taxdump database, attach taxa to profile entries,
    and build/emit the annotated tree
  repo: https://github.com/dkoslicki/TAMPA.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:e2154f1c99a7c986a84a75033443ca2b5b93b66a055d7ba8fa88d414f0238393
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - TAMPA
  - ete3
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# ncbi-taxonomy-resolution

## Summary

Resolves taxIDs from a CAMI-format taxonomic profile against the NCBI taxdump database using the ete3 toolkit's NCBI taxonomy interface, so that per-taxon percentage abundances can be placed on the correct nodes of the taxonomy tree used by TAMPA for visualization.

## When to use

Use when a CAMI-format taxonomic profile contains taxIDs that must be attached to an NCBI taxonomy tree (e.g., to annotate nodes with percentage abundances or emit a Newick tree for downstream plotting).

## When NOT to use

- Input is an amplicon dataset — metacoder-style tools, not TAMPA's NCBI taxdump resolution, are the article's stated wrong fit, and TAMPA is not designed for amplicon data
- You only need a single-value benchmarking metric (e.g., F1 score or UniFrac) — taxonomy-tree construction is unnecessary
- No NCBI taxdump database is available and none can be supplied via -d, since taxa cannot be attached without it

## Inputs

- CAMI-format taxonomic profile file (e.g., mad_yalow_0.profile.txt or gs_marine_short.profile.txt)
- NCBI taxdump database (or a database dump file via -d)
- Taxonomic rank at which to resolve taxa (positional argument, e.g., 'class')

## Outputs

- ete3 taxonomy tree with taxa attached to profile entries and percentage abundances placed on corresponding nodes
- Annotated tree/plot for downstream comparison and visualization

## How to apply

Parse the CAMI-format taxonomic profile (e.g., for sample CAMI_HIGH_S001), extracting taxonomic rank assignments and percentage abundance per taxon. Load the NCBI taxdump database and attach each taxon to its profile entry via the ete3 toolkit's NCBI taxonomy interface, optionally passing a database dump file with the -d/--db_file option. Compute the percentage abundance predictions for each taxon and place them on the corresponding nodes of the taxonomy tree; TAMPA supports a linear scaling key (-k linear) and resolution setting (-r 1600) as in the README example run. Restrict the tree to the input profile's taxIDs with -p/--profile when only profile-derived taxa should be used to construct the tree. Emit the annotated tree via ete3 (saving directly to file with -nm/--no_monitor on headless servers) for downstream comparison or plotting.

## Related tools

- **TAMPA** (Wraps the pipeline that attaches profile taxa to the NCBI taxonomy tree and computes per-taxon percentage abundances) — https://github.com/dkoslicki/TAMPA.git
- **ete3** (Toolkit used to load the NCBI taxdump database, attach taxa to profile entries, and build/emit the annotated tree) — https://github.com/dkoslicki/TAMPA.git

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c False -o .
```

## Evaluation signals

- The output tree file exists and parses as valid Newick
- The tree contains nodes corresponding to the taxIDs present in the input CAMI profile
- Percentage abundances appear on the nodes matching each taxon's rank assignment from the profile
- Running with -p/--profile restricts tree construction to the input profile's taxIDs (no extra taxa from the taxdump appear as leaves)
- Plots render at the requested scaling/resolution (e.g., -k linear -r 1600) and save directly to file when -nm is set

## Limitations

- Building the full taxonomy tree from the taxdump can produce large trees for high-abundance samples; use -thr, -top, or contrast mode (-c True) to condense the visualization
- No taxdump resolution is possible without the NCBI taxdump database or a supplied dump file (-d)
- Headless/server environments require the -nm/--no_monitor flag to save the ete3 tree directly to a file

## Evidence

- [methods] TAMPA's tree-building mechanism takes the computed relative abundances as input and uses the ete3 toolkit: "These relative abundances are then used as input to the tree-building pipeline, which uses the ete3 toolkit to make the desired plot."
- [methods] profile files and NCBI taxdump supply per-taxon abundance predictions: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [readme] -p option restricts tree construction to profile taxIDs: "-p, --profile         specify this option to use only the input profile(s)"
- [readme] -d option specifies the database dump file: "-d DB_FILE, --db_file DB_FILE
                        specify database dump file"
- [readme] -nm flag saves directly to file on monitor-less environments: "-nm, --no_monitor     If you are running on a server or other monitor-less
                        environment, use this flag to save directly to a file"
