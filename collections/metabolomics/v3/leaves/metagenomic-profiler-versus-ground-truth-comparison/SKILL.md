---
name: metagenomic-profiler-versus-ground-truth-comparison
description: Use when benchmarking a taxonomic profiler against a known ground truth
  (e.g. CAMI marmgCAMI2_short_read_sample_0) and you need clade-level presence/absence
  discordance beyond F1 or UniFrac scores.
when_to_use_negative:
- No ground-truth profile is available for the sample — TAMPA's normal (non-contrast)
  mode comparing two or three profilers is the appropriate mode instead
- The goal is amplicon (16S) analysis — metacoder-style tools aside, this skill targets
  WGS metagenomic taxonomic profiles, and amplicon visualization tools are not designed
  for WGS data
- A single aggregate score is sufficient — this skill exists precisely because F1-style
  single metrics miss biological differences, so it adds no value if only a summary
  number is needed
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: TAMPA
  role: 'Primary tool: loads profiler and gold-standard profiles, computes percentage
    relative abundance per taxa, and renders an ete3 tree with contrast-mode false
    positive/negative highlighting'
  repo: https://github.com/dkoslicki/TAMPA.git
- name: ete3
  role: Tree-building toolkit TAMPA uses internally to construct and plot the annotated
    taxonomic tree
- name: pandas
  role: Dependency used for tabulating presence-absence comparisons into a CSV such
    as presence_absence_table.csv
- name: MetaPhyler
  role: Example profiler whose CAMI profile can serve as the input (-i) profile compared
    against the gold standard
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:a95807f675f7f9be4ca09192167ce2660902af35a524cecd9a2f9378a0e44521
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkoslicki/TAMPA.git
  tools:
  - MetaPhyler
  - TAMPA
  - ete3
  - pandas
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metagenomic-profiler-versus-ground-truth-comparison

## Summary

Run TAMPA in contrast mode to compare a taxonomic profiler's output against a gold-standard CAMI2 marine profile at a fixed taxonomic rank (e.g. class/phylum), flagging clades unique to one profile as false positives or false negatives — differences that single-valued metrics such as the F1 score miss.

## When to use

Use when benchmarking a taxonomic profiler against a known ground truth (e.g. CAMI marmgCAMI2_short_read_sample_0) and you need clade-level presence/absence discordance beyond F1 or UniFrac scores.

## When NOT to use

- No ground-truth profile is available for the sample — TAMPA's normal (non-contrast) mode comparing two or three profilers is the appropriate mode instead
- The goal is amplicon (16S) analysis — metacoder-style tools aside, this skill targets WGS metagenomic taxonomic profiles, and amplicon visualization tools are not designed for WGS data
- A single aggregate score is sufficient — this skill exists precisely because F1-style single metrics miss biological differences, so it adds no value if only a summary number is needed

## Inputs

- Profiler taxonomic profile file (e.g. data/mad_yalow_0.profile.txt, MetaPhlAn-style profile)
- Gold-standard taxonomic profile (e.g. data/gs_marine_short.profile.txt)
- Sample identifier to restrict analysis (marmgCAMI2_short_read_sample_0)
- NCBI taxdump database used to create percentage abundance predictions for each taxon

## Outputs

- Taxonomic tree visualization (svg/png/pdf) with false positives and false negatives highlighted in contrast mode
- Clade-level presence-absence comparison table (e.g. presence_absence_table.csv) with discordant calls such as Tenericutes and Planctomycetes
- Optional L1 error plot via -l

## How to apply

Load the profiler output (e.g. data/mad_yalow_0.profile.txt) and the gold-standard profile (data/gs_marine_short.profile.txt) into TAMPA, restricting to a single sample with the -s flag (e.g. marmgCAMI2_short_read_sample_0). Run python src/tampa.py at the class or phylum taxonomic rank with contrast mode enabled (-c True) so clades unique to one profile are highlighted as false positives or false negatives against the gold standard. Pass -p to build the tree from only the input profiles' taxIDs, and optionally -thr or -top to condense large trees to nodes above an abundance threshold or top-abundance nodes. Inspect the resulting annotated tree for discordant clades — in the CAMI2 marine case, the Tenericutes and Planctomycetes phyla are absent from the MetaPhlAn profile but present in the gold standard — and record the 2x2 presence-absence calls. The rationale is that TAMPA computes percentage relative abundance per taxa and identifies which specific clades contributed to metric values, exposing biological divergence that single-value metrics such as F1 miss.

## Related tools

- **TAMPA** (Primary tool: loads profiler and gold-standard profiles, computes percentage relative abundance per taxa, and renders an ete3 tree with contrast-mode false positive/negative highlighting) — https://github.com/dkoslicki/TAMPA.git
- **ete3** (Tree-building toolkit TAMPA uses internally to construct and plot the annotated taxonomic tree)
- **pandas** (Dependency used for tabulating presence-absence comparisons into a CSV such as presence_absence_table.csv)
- **MetaPhyler** (Example profiler whose CAMI profile can serve as the input (-i) profile compared against the gold standard)

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c True -p -o .
```

## Evaluation signals

- The contrast-mode tree marks the Planctomycetes and Tenericutes phylum-level nodes as discordant (absent in MetaPhlAn profile, present in gold standard) on marmgCAMI2_short_read_sample_0
- presence_absence_table.csv contains a row for both Planctomycetes and Tenericutes with a discordant presence-absence call between the two profiles
- The command completes without error using -s marmgCAMI2_short_read_sample_0 and produces a plot matching the README's contrast-mode example figure
- With -c False the same invocation runs in normal mode and highlights abundance differences only when the -dt threshold on percent abundance difference is met, confirming contrast mode is what drives the FP/FN flagging
- Relative abundances in the tree are derived from the profile files plus NCBI taxdump and sum correctly (optionally normalized with -n so node weights sum to one)

## Limitations

- Contrast-mode interpretation was validated on simulated CAMI data where the true ground-truth distribution is known; on real data TAMPA serves hypothesis generation only and findings require follow-up validation
- TAMPA supports up to 3 input profilers at once, so comparisons involving more profilers must be run in batches
- Large trees can be unreadable; -thr, -top, and -dt filtering options are needed to condense the visualization

## Evidence

- [abstract] single metrics, such as the F1 score, fail to capture the biological differences between the datasets: "single metrics, such as the F1 score, fail to capture the biological differences between the datasets"
- [methods] python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0: "python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0"
- [methods] a contrast mode, which highlights the false positives and false negatives: "a contrast mode, which highlights the false positives and false negatives"
- [results] TAMPA does this by computing the percentage relative abundance per taxa and identifying which specific clades contributed to metric values: "TAMPA does this by computing the percentage relative abundance per taxa and identifying which specific clades contributed to metric values"
- [methods] the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
- [readme] TAMPA provides a "CONTRAST MODE" to better visualize the differences between the tool and gold standard: "TAMPA provides a "CONTRAST MODE" to better visualize the differences between the tool and gold standard."
- [discussion] Using simulated data was necessary in order to know the true distribution of the ground truth.: "Using simulated data was necessary in order to know the true distribution of the ground truth."
