---
name: biosynthetic-gene-cluster-genome-mining
description: Use when you have a set of genome assemblies (e.g., 170 S. mutans strains)
  and need to predict specialized-metabolism gene clusters before quantifying their
  abundance in metagenomes.
when_to_use_negative:
- When the target is metagenome-assembled or read-level data directly — this skill
  mines reference genome assemblies; metagenomic reads are only used later by bigmap-map.
- When you need functional MGC quantification per sample — that is the job of downstream
  BiG-MAP mapping/analysis modules, not the mining step.
- When clusters are already redundant-filtered or come from a precomputed GCF set;
  re-running mining and filtering would duplicate work.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0091
tools:
- name: antiSMASH
  role: Predicts biosynthetic gene clusters from genome assemblies; output folders
    are the direct input to bigmap-family
- name: gutSMASH
  role: Alternative cluster predictor for specialized primary metabolism (anaerobic
    MGCs); its output folders can substitute for antiSMASH as bigmap-family input
  repo: https://github.com/victoriapascal/gutsmash
- name: BiG-MAP (bigmap-family)
  role: Performs MinHash-based redundancy filtering and BiG-SCAPE grouping of predicted
    clusters into GCFs and emits the GCF reference files
  repo: https://github.com/medema-group/BiG-MAP
- name: BiG-SCAPE
  role: 'Second redundancy-filtering round: groups BGCs into gene cluster families
    (GCFs) using protein sequence similarity against Pfam-A domains'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:5bfe51d1b6569241bea7e63f1d6966e0574625d63d60595e490c4f3f342b4097
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/victoriapascal/gutsmash
  tools:
  - BiG-MAP
  - BiG-MAP (bigmap-family)
  - BiG-SCAPE
  - antiSMASH
  - bigmap-analyse
  - bigmap-map
  - gutSMASH
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# biosynthetic-gene-cluster-genome-mining

## Summary

Mine bacterial genome assemblies for biosynthetic gene clusters (BGCs/MGCs) with antiSMASH or gutSMASH, then reduce the raw predictions to nonredundant representatives so they can be used as mapping targets downstream. In the source study this turned 170 S. mutans assemblies into 1,849 predicted BGCs, filtered to 41 nonredundant representatives.

## When to use

Use when you have a set of genome assemblies (e.g., 170 S. mutans strains) and need to predict specialized-metabolism gene clusters before quantifying their abundance in metagenomes.

## When NOT to use

- When the target is metagenome-assembled or read-level data directly — this skill mines reference genome assemblies; metagenomic reads are only used later by bigmap-map.
- When you need functional MGC quantification per sample — that is the job of downstream BiG-MAP mapping/analysis modules, not the mining step.
- When clusters are already redundant-filtered or come from a precomputed GCF set; re-running mining and filtering would duplicate work.

## Inputs

- Bacterial genome assemblies in FASTA format (e.g., 170 S. mutans genomes from Tang et al. and Liu et al. cohorts)
- antiSMASH or gutSMASH output folders (one per genome, generated beforehand)
- Processed Pfam-A.hmm database (required for the BiG-SCAPE redundancy-filtering step)

## Outputs

- Per-genome antiSMASH/gutSMASH gene cluster predictions (1,849 BGCs in the source study)
- Nonredundant gene cluster representatives grouped into GCFs (41 representatives in the source study)
- BiG-MAP.GCF.bed (bedfile to extract core regions in bigmap-map)
- BiG-MAP.GCF.fna (reference FASTA for read mapping)
- BiG-MAP.GCs.json and BiG-MAP.GCF.json (gene cluster and GCF dictionaries)

## How to apply

Run antiSMASH (or gutSMASH for specialized primary metabolism) on each genome assembly so that each genome yields an output folder of predicted gene clusters, as done for the 170 S. mutans 14D-related genomes that produced 1,849 BGCs. Feed the collection of anti-/gutSMASH output directories into BiG-MAP's bigmap-family module, which first performs MinHash (MASH)-based redundancy filtering with a default sequence-similarity cut-off of 0.8 to avoid ambiguous read mapping. Optionally run a second, more accurate redundancy-filtering round with BiG-SCAPE, which requires the latest processed Pfam-A.hmm database (compressed file from Interpro, uncompressed and pressed with hmmpress). Control grouping granularity with bigmap-family's -tg threshold (default 0.8; fraction at which cluster protein sequences are considered similar, 0 = every cluster its own family, 1 = one giant family) and -th (default 0.1) for housekeeping genes. The module outputs BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json, and BiG-MAP.GCF.json, which become the mapping references for bigmap-map.

## Related tools

- **antiSMASH** (Predicts biosynthetic gene clusters from genome assemblies; output folders are the direct input to bigmap-family)
- **gutSMASH** (Alternative cluster predictor for specialized primary metabolism (anaerobic MGCs); its output folders can substitute for antiSMASH as bigmap-family input) — https://github.com/victoriapascal/gutsmash
- **BiG-MAP (bigmap-family)** (Performs MinHash-based redundancy filtering and BiG-SCAPE grouping of predicted clusters into GCFs and emits the GCF reference files) — https://github.com/medema-group/BiG-MAP
- **BiG-SCAPE** (Second redundancy-filtering round: groups BGCs into gene cluster families (GCFs) using protein sequence similarity against Pfam-A domains)

## Examples

```
bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- Number of predicted BGCs scales with genome count: 170 S. mutans genomes yielded 1,849 BGCs; large deviations suggest misconfigured antiSMASH runs or missing genome folders.
- After MinHash (default 0.8 similarity cut-off) and BiG-SCAPE filtering, the 1,849 clusters reduce to 41 nonredundant representatives; far fewer suggests thresholds set too low (e.g., -tg near 0 gives no reduction benefit) or input folders missing clusters.
- bigmap-family must emit all four artifacts (BiG-MAP.GCF.bed, BiG-MAP.GCF.fna, BiG-MAP.GCs.json, BiG-MAP.GCF.json); missing files break the downstream bigmap-map bed extraction and mapping.
- GCF membership in BiG-MAP.GCF.json should match BiG-SCAPE family assignments, and the muc operon from S. mutans 14D must be present as a recoverable representative for the oral-caries analysis.

## Limitations

- BiG-MAP is only tested on Ubuntu per the README; other platforms may need workarounds.
- BiG-SCAPE redundancy filtering is optional but strongly recommended — without it, ambiguous read mapping to redundant clusters can inflate abundance estimates.
- The -tg/-th thresholds change clustering granularity: -tg=0 puts every cluster in its own family, -tg=1 merges all clusters into one family, so default 0.8 must be justified for the organism under study.
- antiSMASH/gutSMASH must be run beforehand; bigmap-family does not perform cluster prediction itself.

## Evidence

- [intro] Cluster mining tools enumerated: "Several tools have been developed to mine genomes for these gene clusters, including antiSMASH (10), gutSMASH (11), and DeepBGC (12)."
- [intro] GCF grouping rationale: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
- [readme] bigmap-family input requirement: [evidence span withheld — anchor status: external_doc]
- [readme] MASH redundancy-filtering default threshold: [evidence span withheld — anchor status: external_doc]
- [readme] Pfam requirement for BiG-SCAPE step: [evidence span withheld — anchor status: external_doc]
