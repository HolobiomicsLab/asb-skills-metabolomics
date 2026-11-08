---
name: clade-level-false-positive-negative-identification
description: Use when two taxonomic profiles (a profiler output and a ground truth)
  receive similar single-value metric scores but you need to identify which specific
  taxonomic clades differ between them.
when_to_use_negative:
- No ground-truth taxonomic profile is available — TAMPA's contrast mode requires
  a gold standard to call false positives/false negatives.
- You only need a single aggregate metric (F1, UniFrac) for profiler benchmarking;
  this skill targets clade-level biological differences those metrics miss.
- The input is amplicon (16S) data — the article notes metacoder-style amplicon tools
  are separate, and TAMPA targets WGS metagenomic profiles.
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: TAMPA
  role: Runs the contrast-mode comparison of profiler vs. gold-standard profiles and
    renders the annotated taxonomic tree
  repo: https://github.com/dkoslicki/TAMPA.git
- name: ete3
  role: Toolkit used by TAMPA's tree-building pipeline to construct and plot the taxonomic
    tree
  repo: https://github.com/dkoslicki/TAMPA.git
- name: pandas
  role: Dependency used to assemble the clade presence-absence table output as CSV
  repo: https://github.com/dkoslicki/TAMPA.git
- name: OPAL
  role: Benchmarking platform into which TAMPA is integrated for taxonomic profiler
    assessment
  repo: https://github.com/CAMI-challenge/firstchallenge_evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad008
    title: 'TAMPA: interpretable analysis and visualization of metagenomics-based
      taxon abundance profiles'
  source_content_hash: sha256:ba664eaa4e80dca302031de554065c7e47d237779af8c5e74561a235c3269e3f
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
yes_rate: 0.5
n_claims: 8
---

# clade-level-false-positive-negative-identification

## Summary

Applies TAMPA's contrast mode to a profiler output versus a gold-standard CAMI2 profile at the class/phylum rank, flagging clades unique to one profile as false positives or false negatives. This exposes biological divergences (e.g., missed Tenericutes and Planctomycetes) that single-value metrics like the F1 score hide.

## When to use

Use when two taxonomic profiles (a profiler output and a ground truth) receive similar single-value metric scores but you need to identify which specific taxonomic clades differ between them.

## When NOT to use

- No ground-truth taxonomic profile is available — TAMPA's contrast mode requires a gold standard to call false positives/false negatives.
- You only need a single aggregate metric (F1, UniFrac) for profiler benchmarking; this skill targets clade-level biological differences those metrics miss.
- The input is amplicon (16S) data — the article notes metacoder-style amplicon tools are separate, and TAMPA targets WGS metagenomic profiles.

## Inputs

- Profiler taxonomic profile file (e.g., MetaPhlAn output data/mad_yalow_0.profile.txt)
- Gold-standard taxonomic profile (data/gs_marine_short.profile.txt from the CAMI2 marine dataset)
- Sample identifier (marmgCAMI2_short_read_sample_0)
- NCBI taxdump database for taxID tree construction

## Outputs

- TAMPA contrast-mode tree plot highlighting false-positive and false-negative clades
- 2x2 presence-absence table of clade calls (presence_absence_table.csv) with discordant Planctomycetes and Tenericutes rows

## How to apply

Load the MetaPhlAn profile (data/mad_yalow_0.profile.txt) and the gold-standard marine profile (data/gs_marine_short.profile.txt) with TAMPA, restricting to sample marmgCAMI2_short_read_sample_0 via the -s flag. Run python src/tampa.py at the class or phylum taxonomic rank with contrast mode enabled (-c True) so clades unique to one profile are flagged as false positives or false negatives against the gold standard. Pass -p to build the tree from only the input profile taxIDs, and optionally set -dt so taxa whose abundance differs by a given percentage are highlighted. Inspect the Planctomycetes and Tenericutes phylum-level nodes in the comparison, record each clade as present or absent in each profile, and write the 2x2 presence-absence table to presence_absence_table.csv.

## Related tools

- **TAMPA** (Runs the contrast-mode comparison of profiler vs. gold-standard profiles and renders the annotated taxonomic tree) — https://github.com/dkoslicki/TAMPA.git
- **ete3** (Toolkit used by TAMPA's tree-building pipeline to construct and plot the taxonomic tree) — https://github.com/dkoslicki/TAMPA.git
- **pandas** (Dependency used to assemble the clade presence-absence table output as CSV) — https://github.com/dkoslicki/TAMPA.git
- **OPAL** (Benchmarking platform into which TAMPA is integrated for taxonomic profiler assessment) — https://github.com/CAMI-challenge/firstchallenge_evaluation

## Examples

```
python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0 -b marine_test -k linear -r 1600 -c True -o .
```

## Evaluation signals

- presence_absence_table.csv contains both Planctomycetes and Tenericutes rows with discordant presence-absence calls between the MetaPhlAn profile and the gold standard.
- Contrast-mode run completes without error and the rendered tree flags the unique-to-one-profile clades as false positives/false negatives.
- Taxonomic rank argument (class or phylum) resolves the Planctomycetes and Tenericutes phylum-level nodes in the ete3 tree.
- The -s filter restricts output to marmgCAMI2_short_read_sample_0, matching the CAMI2 marine sample of interest.
- Tree node abundances derive from percentage relative abundances computed from the profile files and the NCBI taxdump database.

## Limitations

- Hypotheses generated by TAMPA on real biological data require further analysis and follow-up studies for validation.
- Validation was limited to simulated data; the true ground-truth distribution is only known for simulated data.
- Contrast mode requires a ground truth profile, so it cannot be applied to profiler-vs-profiler comparisons lacking gold standards; the -thr and -top options may be needed to condense very large trees.

## Evidence

- [methods] contrast mode highlighting false positives and false negatives: "a contrast mode, which highlights the false positives and false negatives"
- [abstract] Single metrics miss biological differences: "single metrics, such as the F1 score, fail to capture the biological differences between the datasets"
- [readme] Contrast mode README: "-c: Contrast mode can be used to identify problematic subregions of large trees. In the contrast mode, the false positive taxas are represented as"
- [methods] Marine dataset invocation: "python src/tampa.py -i data/mad_yalow_0.profile.txt -g data/gs_marine_short.profile.txt class -s marmgCAMI2_short_read_sample_0"
- [discussion] Real-data validation caveat: "While TAMPA can be used for hypothesis generation on real biological data, further analysis and follow-up studies will be required to validate the hypotheses generated by TAMPA."
