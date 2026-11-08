---
name: tampa-contrast-mode-configuration
description: Use when you have a taxonomic profiler output and a ground-truth profile
  for the same sample (e.g., CAMI2 marine marmgCAMI2_short_read_sample_0) and need
  to find which specific clades a high-scoring profiler misses.
when_to_use_negative:
- Input is amplicon microbial data — tools like metacoder are for amplicon data, but
  TAMPA is designed for WGS metagenomic profiles; conversely metacoder should not
  be substituted here.
- No ground-truth profile exists for the sample — contrast mode requires a gold standard
  to flag false positives/false negatives; without one use TAMPA's normal mode to
  compare profiler outputs.
- Goal is a single-value benchmarking score rather than identifying specific discordant
  clades — use a benchmarking platform like OPAL for aggregate metrics.
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: TAMPA
  role: Main tool run with -c True contrast mode to highlight false positive and false
    negative taxa against a gold standard on a taxonomic tree
  repo: https://github.com/dkoslicki/TAMPA.git
- name: ete3
  role: Toolkit used by TAMPA's tree-building pipeline to render the profile comparison
    tree
  repo: https://github.com/dkoslicki/TAMPA.git
- name: pandas
  role: Dependency used to assemble and write the per-clade presence-absence comparison
    table (CSV)
  repo: https://github.com/dkoslicki/TAMPA.git
- name: OPAL
  role: Taxonomic profiling benchmarking platform into which TAMPA can be incorporated
    for integrated benchmarking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:7d88fd8cf74a45ca3fdf1afb40feef6cfd8b16e7ee04de0fbec931cf86a82e09
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
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# tampa-contrast-mode-configuration

## Summary

Configure and run TAMPA in contrast mode (-c True) to compare a profiler's taxonomic profile against a gold-standard profile at a chosen taxonomic rank, highlighting clades present in only one profile (false positives/false negatives) that single-value metrics such as the F1 score miss.

## When to use

Use when you have a taxonomic profiler output and a ground-truth profile for the same sample (e.g., CAMI2 marine marmgCAMI2_short_read_sample_0) and need to find which specific clades a high-scoring profiler misses.

## When NOT to use

- Input is amplicon microbial data — tools like metacoder are for amplicon data, but TAMPA is designed for WGS metagenomic profiles; conversely metacoder should not be substituted here.
- No ground-truth profile exists for the sample — contrast mode requires a gold standard to flag false positives/false negatives; without one use TAMPA's normal mode to compare profiler outputs.
- Goal is a single-value benchmarking score rather than identifying specific discordant clades — use a benchmarking platform like OPAL for aggregate metrics.

## Inputs

- Profiler taxonomic profile file (e.g., data/mad_yalow_0.profile.txt)
- Gold-standard taxonomic profile file (e.g., data/gs_marine_short.profile.txt)
- NCBI taxdump database (for percentage abundance predictions per taxon)
- Sample identifier to restrict analysis (e.g., marmgCAMI2_short_read_sample_0)
- Taxonomic rank to plot at (e.g., class, phylum)

## Outputs

- Phylogenetic tree visualization highlighting false positive and false negative taxa in contrast mode
- Presence-absence calls per clade comparing profiler vs. gold standard (e.g., a 2x2 table such as presence_absence_table.csv)

## How to apply

Load the profiler profile (e.g., data/mad_yalow_0.profile.txt) and the gold-standard profile (e.g., data/gs_marine_short.profile.txt) with TAMPA, restricting to a single sample via the -s flag (e.g., marmgCAMI2_short_read_sample_0). Run python src/tampa.py with the taxonomic rank (e.g., class or phylum) as the positional argument and enable contrast mode with -c True so clades unique to one profile are flagged as false positives or false negatives against the gold standard. Use the -p option to construct the tree from only the input profile taxIDs when a condensed comparison is needed. The rationale is that single metrics like the F1 score fail to capture biological differences between datasets; contrast mode surfaces the discordant clades (e.g., phylum-level Planctomycetes and Tenericutes absent from MetaPhlAn's profile but present in the gold standard). Inspect the resulting tree for highlighted false positive/false negative subregions and record each clade's presence/absence per profile.

## Related tools

- **TAMPA** (Main tool run with -c True contrast mode to highlight false positive and false negative taxa against a gold standard on a taxonomic tree) — https://github.com/dkoslicki/TAMPA.git
- **ete3** (Toolkit used by TAMPA's tree-building pipeline to render the profile comparison tree) — https://github.com/dkoslicki/TAMPA.git
- **pandas** (Dependency used to assemble and write the per-clade presence-absence comparison table (CSV)) — https://github.com/dkoslicki/TAMPA.git
- **OPAL** (Taxonomic profiling benchmarking platform into which TAMPA can be incorporated for integrated benchmarking)

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c True -o .
```

## Evaluation signals

- Presence-absence table contains discordant rows for Planctomycetes and Tenericutes (phylum level): present in the gold standard (gs_marine_short.profile.txt) and absent from the MetaPhlAn profile (mad_yalow_0.profile.txt) for marmgCAMI2_short_read_sample_0.
- The contrast-mode run flags clades unique to one profile as false positives or false negatives rather than only displaying relative abundances (normal mode behavior).
- The generated tree restricts nodes per sample via the -s flag and, if -p is used, contains only taxIDs present in the input profiles.
- The biological divergence found is invisible to single-value metrics — the sample's F1-style score can remain high despite the missed phyla, confirming the finding is a clade-level result.
- Running the command without -c True (contrast False) produces the normal-mode relative abundance plot instead of the false positive/false negative highlight, providing a negative control.

## Limitations

- TAMPA was validated on simulated data (e.g., CAMI) where the ground truth is known; on real biological data TAMPA serves hypothesis generation and hypotheses require further follow-up validation.
- Large trees may be hard to read; contrast mode and filtering options like -thr and -top exist specifically to identify problematic subregions or condense the display.
- Contrast mode supports comparison of a profile against a ground truth; up to 3 input profilers are supported in normal mode, not as a contrast-mode gold-standard comparison.

## Evidence

- [readme] Contrast mode can be used to identify problematic subregions of large trees. In the contrast mode, the false positive taxas are represented as: "Contrast mode can be used to identify problematic subregions of large trees."
- [abstract] single metrics, such as the F1 score, fail to capture the biological differences between the datasets: "single metrics, such as the F1 score, fail to capture the biological differences between the datasets"
- [methods] python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0: "python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0"
- [methods] These relative abundances are then used as input to the tree-building pipeline, which uses the ete3 toolkit to make the desired plot.: "the profile files and the NCBI taxdump database are used to create the percentage abundance predictions for each taxon"
