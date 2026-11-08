---
name: operon-redundancy-removal
description: Use when multiple per-sample operon prediction files (e.g. MetaRon Operon_File.tab
  outputs from metagenomic samples) must be merged into one non-redundant operon set
  for cataloguing or downstream abundance analysis.
when_to_use_negative:
- Input is a single sample or single-genome whole-genome operon prediction — redundancy
  removal across samples is a metagenomic cataloguing step, not needed for one genome's
  predictions
- Operons already carry functional annotation (e.g. pre-clustered KEGG-linked operon
  sets); MetaRon is designed for prediction without functional information, so re-deriving
  operons here would duplicate existing structure
- Reads are unfiltered or unassembled and only a gene list is needed — run MetaRon
  assembly/gene-prediction first; deduplication alone will not produce operons from
  FASTQ
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaRon
  role: Pipeline that assembles reads, predicts genes and operons per sample, and
    emits the Operon_File.tab whose records are deduplicated across samples
  repo: https://github.com/zaidissa/MetaRon
- name: IDBA
  role: De novo assembler used inside MetaRon 'ago' mode to build scaftigs from paired-end
    FASTQ reads prior to operon prediction
- name: prodigal
  role: Gene prediction tool (--tool 2) used on filtered scaftigs; predicted genes
    are the units clustered into operons
- name: NNPP
  role: Neural network promoter prediction (NNPP2.2, path configured in config.txt)
    supporting de novo promoter/proximon prediction used in operon calling
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:691a163f7ce250a28ca9c7bd49ce5fe41a8da8195f759207010ac1614e926a36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - IDBA
  - MetaRon
  - NNPP
  - prodigal
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# operon-redundancy-removal

## Summary

Deduplicates per-sample operon predictions from MetaRon runs into a single non-redundant metagenomic operon catalogue, as applied to 145 paired-end human gut metagenome samples to yield 1,232,407 unique operons.

## When to use

Use when multiple per-sample operon prediction files (e.g. MetaRon Operon_File.tab outputs from metagenomic samples) must be merged into one non-redundant operon set for cataloguing or downstream abundance analysis.

## When NOT to use

- Input is a single sample or single-genome whole-genome operon prediction — redundancy removal across samples is a metagenomic cataloguing step, not needed for one genome's predictions
- Operons already carry functional annotation (e.g. pre-clustered KEGG-linked operon sets); MetaRon is designed for prediction without functional information, so re-deriving operons here would duplicate existing structure
- Reads are unfiltered or unassembled and only a gene list is needed — run MetaRon assembly/gene-prediction first; deduplication alone will not produce operons from FASTQ

## Inputs

- Per-sample MetaRon Operon_File.tab files (tab-delimited operon predictions, one per sample)
- Per-sample scaftig FASTA assemblies and gene prediction .tab files (produced via 'ago' mode with IDBA and Prodigal/MetaGeneMark)
- Paired-end whole-metagenome shotgun FASTQ read pairs (e.g. 145 human gut samples from SRP008047)

## Outputs

- Non-redundant metagenomic operon catalogue (1,232,407 unique operons for the 145 human gut samples)
- Concatenated tab-delimited Operon_File.tab summarizing unique operons across samples

## How to apply

First generate per-sample operon predictions by running MetaRon on each sample: in 'ago' mode, raw paired-end FASTQ reads are de novo assembled into scaftigs via IDBA, scaftigs ≤ 500 bp are removed, genes are predicted with MetaGeneMark (tool 1) or Prodigal (tool 2), and co-directional proximal genes with intergenic distance (IGD) < 601 bp are clustered into operons after de novo promoter prediction with NNPP2.2. Each sample yields a tab-delimited Operon_File.tab. Then remove operonic redundancy across all samples — identical operon calls (same gene composition on the same assembled sequence context) found in multiple samples are collapsed to a single unique operon entry. Concatenate the deduplicated records into a combined catalogue; for the 145 human gut samples (study SRP008047) this catalogue must contain 1,232,407 unique operons. Rationale: per-sample prediction inflates counts because the same genomic regions recur across individuals, so redundancy removal is required before any abundance or secondary-metabolite association analysis (e.g. type 2 diabetes associations with K00691, K21279, K07101).

## Related tools

- **MetaRon** (Pipeline that assembles reads, predicts genes and operons per sample, and emits the Operon_File.tab whose records are deduplicated across samples) — https://github.com/zaidissa/MetaRon
- **IDBA** (De novo assembler used inside MetaRon 'ago' mode to build scaftigs from paired-end FASTQ reads prior to operon prediction)
- **prodigal** (Gene prediction tool (--tool 2) used on filtered scaftigs; predicted genes are the units clustered into operons)
- **NNPP** (Neural network promoter prediction (NNPP2.2, path configured in config.txt) supporting de novo promoter/proximon prediction used in operon calling)

## Examples

```
metaron --sample ERR022075 --process ago --read_type merge --read_length r --paired_1 ERR022075.1.fastq --paired_2 ERR022075.2.fastq --output ~/out/
```

## Evaluation signals

- Concatenated, deduplicated catalogue for the 145 human gut samples (SRP004969-class SRP008047 data) contains exactly 1,232,407 unique operons, matching the paper's reported count
- Every retained operon satisfies the co-directional proximon criterion: genes on the same strand with IGD < 601 bp and promoter evidence from NNPP2.2
- No scaftig ≤ 500 bp contributed genes to the final catalogue (500 bp scaftig filter applied upstream)
- Unique operon count across the catalogue is strictly less than the sum of per-sample operon predictions, confirming redundancy was actually removed
- Operon_File.tab remains tab-delimited with the schema MetaRon documents for operonic information per record

## Limitations

- Redundancy removal was performed separately on predicted operons rather than being integrated into the MetaRon pipeline; downstream analysis is manual/separate
- Secondary metabolite prediction, pathway annotation and graphical representation are not integrated (planned future work), so the deduplicated catalogue alone yields no functional trends without external tools (antiSMASH, FMAP)
- Pipeline is restricted to Linux and academic use only; NNPP2.2 path must be manually configured in config.txt, and a promoter-prediction failure means NNPP cannot identify the promoter
- Python version conflict between paper (Python > 3.0) and README (Python 2.7) should be checked before reproducing

## Evidence

- [abstract] 1,232,407 unique operons were identified from 145 paired-end human gut metagenome samples: "Finally, we identified 1,232,407 unique operons from 145 paired-end human gut metagenome samples."
- [results] Removing operonic redundancy produced 1.23 million unique operons: "Removing operonic redundancy produced 1.23 million unique operons."
- [results] IGD < 601 bp threshold for co-directional proximal gene clusters: "MetaRon identified 822 co-directional proximal gene clusters (IGD < 601 bp), containing 2955 genes."
- [results] 500 bp scaftig length filter: "Scaftigs with length less than or equal to 500 bp were removed."
- [readme] Operon prediction output file from README: "This will save metagenomic operon predictions ``Operon_File.tab``.  The prediction file will report the operonic information based on the above mentioned parameters."
