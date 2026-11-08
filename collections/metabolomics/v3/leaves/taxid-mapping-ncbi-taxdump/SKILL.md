---
name: taxid-mapping-ncbi-taxdump
description: Use when profiler output profiles carry numeric taxIDs that must be mapped
  to NCBI taxonomy names/lineages and converted to percentage relative abundances
  per taxon before ranking profilers or drawing abundance trees.
when_to_use_negative:
- Amplicon microbial data — metacoder-style amplicon tools are the wrong fit and this
  WGS-oriented taxdump mapping is designed for metagenomic taxonomic profiles
- Profiles that already carry full lineage strings and precomputed relative abundances
  and need no taxID-to-taxon resolution or tree placement
- Analyses on real biological data with no ground truth where the goal is validated
  performance ranking rather than hypothesis generation
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0699
tools:
- name: TAMPA
  role: Consumes profiler profiles plus the NCBI taxdump to build percentage abundance
    predictions per taxon and render rank-specific abundance trees
  repo: https://github.com/dkoslicki/TAMPA.git
- name: ete3
  role: Tree-building toolkit used by TAMPA to place the taxdump-mapped relative abundances
    on the taxonomy tree
- name: NCBI taxdump
  role: Reference taxonomy database against which profile taxIDs are resolved to lineages
    and rank names
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:40335828969bc635ed4d4cfe78184cb08238ad14adb48b10c6cfff2d63c8a69d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - NCBI taxdump
  - TAMPA
  - ete3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0699
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# taxid-mapping-ncbi-taxdump

## Summary

Resolve NCBI taxonomy identifiers from standardized taxonomic profiler outputs (e.g., CAMI-style profile files) against the NCBI taxdump database to build per-taxon percentage relative abundances and lineage-annotated trees, as done in the TAMPA pipeline.

## When to use

Use when profiler output profiles carry numeric taxIDs that must be mapped to NCBI taxonomy names/lineages and converted to percentage relative abundances per taxon before ranking profilers or drawing abundance trees.

## When NOT to use

- Amplicon microbial data — metacoder-style amplicon tools are the wrong fit and this WGS-oriented taxdump mapping is designed for metagenomic taxonomic profiles
- Profiles that already carry full lineage strings and precomputed relative abundances and need no taxID-to-taxon resolution or tree placement
- Analyses on real biological data with no ground truth where the goal is validated performance ranking rather than hypothesis generation

## Inputs

- Standardized taxonomic profiler output profiles containing taxIDs (e.g., MetaPhyler, mOTU, Taxy-Pro submissions from CAMI-challenge/firstchallenge_evaluation profile_submissions)
- Ground-truth taxonomic profile (e.g., gold standard for CAMI_HIGH_S001)
- NCBI taxdump database (nodes.dmp/names.dmp dump files)

## Outputs

- Percentage relative abundance per taxon (taxID mapped to NCBI lineage)
- Taxonomy tree plot (svg/png/pdf) with abundances placed at the chosen taxonomic rank
- Optional per-profiler L1 error values when run with -l/--plot_l1

## How to apply

Load each profiler's output profile together with the NCBI taxdump database; in TAMPA these are consumed to 'create the percentage abundance predictions for each taxon' and then compared across tools to compute relative abundances. Supply the taxdump dump via the -d/--db_file option and pick the target taxonomic rank as the positional argument (e.g., class), optionally applying -n/--normalize so node weights/relative abundances sum to one. The mapped relative abundances feed the ete3-based tree-building pipeline, which places abundances on the NCBI taxonomy tree at the requested rank; taxIDs absent from the taxdump cannot be placed and will be dropped from the visualization. Correctness is judged by whether each taxID resolves to the expected lineage, abundances are expressed as percentages (summing to ~100%, or 1 when normalized), and the resulting tree matches the ground-truth profile's taxonomy (e.g., CAMI_HIGH_S001 gold standard).

## Related tools

- **TAMPA** (Consumes profiler profiles plus the NCBI taxdump to build percentage abundance predictions per taxon and render rank-specific abundance trees) — https://github.com/dkoslicki/TAMPA.git
- **ete3** (Tree-building toolkit used by TAMPA to place the taxdump-mapped relative abundances on the taxonomy tree)
- **NCBI taxdump** (Reference taxonomy database against which profile taxIDs are resolved to lineages and rank names)

## Examples

```
python src/tampa.py -i tool.profile -g ground_truth.profile class -s CAMI_HIGH_S001 -b cami_high_s001 -k linear -r 1600 -n -d taxdump.sqlite -o .
```

## Evaluation signals

- Every taxID in the input profile resolves in the taxdump to a lineage containing the requested rank (e.g., 'class'); unresolvable taxIDs are identifiable/dropped
- Percentage relative abundances per taxon sum to ~100 (or to 1 when -n/--normalize is applied)
- Mapped abundances reproduce the same per-taxon values when recomputed against the ground-truth profile for the same sample (e.g., CAMI_HIGH_S001)
- Resulting tree plot places taxa under the correct NCBI parent nodes and matches the figure produced by the documented example command
- Downstream L1-error ranking (lowest error = top performer, e.g., MetaPhyler over mOTU) is consistent with the paper's reported CAMI result

## Limitations

- Ground-truth distributions are only truly known for simulated data; on real data the mapping supports hypothesis generation, and further analysis/follow-up studies are required for validation
- TaxIDs not present in the supplied taxdump cannot be mapped or visualized
- Large trees can be unwieldy; condense with -thr (abundance threshold) or -top before interpreting mapped abundances

## Evidence

- [methods] the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [methods] tree pipeline: "These relative abundances are then used as input to the tree-building pipeline, which uses the ete3 toolkit to make the desired plot."
- [readme] db_file option: "-d DB_FILE, --db_file DB_FILE
                        specify database dump file"
- [readme] normalize option: "-n, --normalize       specify this option if you want to normalize the node
                        weights/relative abundances so that they sum to one"
- [methods] CAMI sample scope: "We limited our analysis to sample 1 of the high-complexity dataset, denoted by CAMI_HIGH_S001."
