---
name: phylum-level-taxonomic-clade-inspection
description: Use when comparing a taxonomic profiler's output against a ground-truth
  CAMI profile and you need to identify which specific clades (e.g., at class/phylum
  rank) are discordantly present or absent between the two profiles.
when_to_use_negative:
- Input is a single profiler profile with no gold-standard comparison available —
  contrast mode and false-positive/false-negative calling require a ground-truth profile
- The analysis goal is a global single-value benchmark score (e.g., F1, UniFrac) rather
  than clade-level biological interpretation — use a benchmarking platform like OPAL
  instead
- Amplicon (16S) microbial data — TAMPA's WGS metagenomic profile visualization is
  not designed for amplicon analyses (cf. metacoder's limitation)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: TAMPA
  role: 'Primary tool: computes percentage relative abundance per taxon, builds an
    ete3 tree, and uses contrast mode to highlight false-positive and false-negative
    clades versus the gold standard'
  repo: https://github.com/dkoslicki/TAMPA.git
- name: ete3
  role: Toolkit underlying TAMPA's tree-building pipeline for visualizing relative
    abundances on the taxonomic tree
- name: pandas
  role: Used to assemble and export the clade presence-absence table (presence_absence_table.csv)
    from the TAMPA comparison
- name: MetaPhyler
  role: Example CAMI profiler whose output profile is compared against the gold standard;
    here represented by the MetaPhlAn profile mad_yalow_0.profile.txt
- name: OPAL
  role: Taxonomic profiling benchmarking platform into which TAMPA is integrated;
    provides the single-value metric context that this clade inspection complements
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:a6454076a8ff310b72a387b06aa46bbfbd4c6e8f1a07f7044479c9d09bf049b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - MetaPhyler
  - OPAL
  - TAMPA
  - ete3
  - pandas
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# phylum-level-taxonomic-clade-inspection

## Summary

Inspect phylum/class-level clades in a taxonomic profiler output versus a gold-standard CAMI2 marine profile using TAMPA's contrast mode, to surface biological divergences (e.g., missed Planctomycetes and Tenericutes) that single-value metrics such as the F1 score cannot reveal.

## When to use

Use when comparing a taxonomic profiler's output against a ground-truth CAMI profile and you need to identify which specific clades (e.g., at class/phylum rank) are discordantly present or absent between the two profiles.

## When NOT to use

- Input is a single profiler profile with no gold-standard comparison available — contrast mode and false-positive/false-negative calling require a ground-truth profile
- The analysis goal is a global single-value benchmark score (e.g., F1, UniFrac) rather than clade-level biological interpretation — use a benchmarking platform like OPAL instead
- Amplicon (16S) microbial data — TAMPA's WGS metagenomic profile visualization is not designed for amplicon analyses (cf. metacoder's limitation)

## Inputs

- Profiler taxonomic profile file (e.g., MetaPhlAn output data/mad_yalow_0.profile.txt)
- Gold-standard taxonomic profile (data/gs_marine_short.profile.txt from CAMI2 marine dataset)
- Sample identifier of interest (marmgCAMI2_short_read_sample_0)
- NCBI taxdump database for taxon tree construction

## Outputs

- TAMPA phylogenetic tree visualization highlighting false-positive and false-negative clades at the class/phylum rank
- Presence-absence table (presence_absence_table.csv) with discordant clade calls (e.g., Planctomycetes, Tenericutes) between profiler and gold standard

## How to apply

Load the profiler output (e.g., data/mad_yalow_0.profile.txt) and the gold-standard marine profile (data/gs_marine_short.profile.txt) into TAMPA, restricting to sample marmgCAMI2_short_read_sample_0 with the -s flag. Run python src/tampa.py at the class taxonomic rank with contrast mode enabled (-c True) so clades unique to one profile are flagged as false positives or false negatives against the gold standard. Optionally use -p to build the tree from only the input profiles' taxIDs. Inspect the phylum-level nodes (e.g., Planctomycetes and Tenericutes) in the resulting comparison, record each clade's presence/absence in both profiles, and write the 2x2 presence-absence table (e.g., presence_absence_table.csv). The rationale is that TAMPA's contrast mode highlights false positives and false negatives, exposing biological differences invisible to single-value metrics like the F1 score.

## Related tools

- **TAMPA** (Primary tool: computes percentage relative abundance per taxon, builds an ete3 tree, and uses contrast mode to highlight false-positive and false-negative clades versus the gold standard) — https://github.com/dkoslicki/TAMPA.git
- **ete3** (Toolkit underlying TAMPA's tree-building pipeline for visualizing relative abundances on the taxonomic tree)
- **pandas** (Used to assemble and export the clade presence-absence table (presence_absence_table.csv) from the TAMPA comparison)
- **MetaPhyler** (Example CAMI profiler whose output profile is compared against the gold standard; here represented by the MetaPhlAn profile mad_yalow_0.profile.txt)
- **OPAL** (Taxonomic profiling benchmarking platform into which TAMPA is integrated; provides the single-value metric context that this clade inspection complements)

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c True -p -o .
```

## Evaluation signals

- presence_absence_table.csv contains rows for both Planctomycetes and Tenericutes with discordant presence-absence calls between the MetaPhlAn profile and the gold standard
- Contrast-mode tree flags the Tenericutes and Planctomycetes phylum nodes as false negatives (absent from the profiler output but present in the ground truth)
- Running TAMPA without contrast mode does not surface the same discordance, confirming the finding depends on the -c True comparison
- The gold-standard marine profile (gs_marine_short.profile.txt) contains nonzero abundance for both phyla at sample marmgCAMI2_short_read_sample_0, while the profiler profile contains neither
- The divergence is invisible to single-valued metrics (e.g., high F1 score), so any metric-based summary alone cannot reproduce the table

## Limitations

- Ground truth is only available for simulated datasets like CAMI; on real biological data TAMPA supports hypothesis generation, and further follow-up studies are required to validate the generated hypotheses
- The result is specific to the profiled sample and profiler; a different profiler (e.g., mOTU, Taxy-Pro) or sample may show a different pattern of missed clades
- Large trees can obscure problematic subregions; contrast mode and threshold/top filters (-thr, -top) are needed to condense the visualization

## Evidence

- [methods] contrast mode, which highlights the false positives and false negatives: "a contrast mode, which highlights the false positives and false negatives"
- [abstract] TAMPA clade-specific differences: "single metrics, such as the F1 score, fail to capture the biological differences between the datasets"
- [readme] Contrast mode CLI example: "python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c True -o ."
- [readme] Contrast mode rationale for large trees: "Contrast mode can be used to identify problematic subregions of large trees."
- [discussion] Hypotheses require follow-up validation: "While TAMPA can be used for hypothesis generation on real biological data, further analysis and follow-up studies will be required to validate the hypotheses generated by TAMPA."
