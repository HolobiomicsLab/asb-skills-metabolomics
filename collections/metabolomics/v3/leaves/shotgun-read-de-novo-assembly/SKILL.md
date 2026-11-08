---
name: shotgun-read-de-novo-assembly
description: Use when starting from raw paired-end Illumina FASTQ shotgun reads (simulated
  via NeSSM or real, e.g. SRP029211) and scaftigs are needed as input for gene/operon
  prediction without a reference genome.
when_to_use_negative:
- Scaffolds and gene predictions are already available — use MetaRon's 'op' mode directly
  on scaftigs and the gene .tab file instead of reassembling
- Input is not shotgun reads but long/reference-quality genomes where reference-based
  operon annotation (e.g. DOOR/ODB/RegulonDB) is available
- Reads are not filtered or are single-end in an unsupported layout, violating MetaRon's
  paired-end FASTQ input requirements
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaRon
  role: Command-line pipeline wrapper that orchestrates the assembly ('ago' mode)
    and passes scaftigs to gene and operon prediction
  repo: github.com/zaidissa/MetaRon
- name: IDBA
  role: Iterative De Bruijn Graph de novo assembler that builds scaftigs from the
    paired-end shotgun reads
- name: NeSSM
  role: Simulator used to generate the two million paired-end Illumina reads at 15X
    depth from the three reference genomes for the simulated metagenome
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:8f337becd1cf9b1b3924243f1887be23403fff9e12325eeebd087eb55ab5b9d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IDBA
  - MetaRon
  - NNPP
  - NeSSM
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# shotgun-read-de-novo-assembly

## Summary

De novo assembly of filtered shotgun (paired-end) metagenomic or whole-genome reads into scaftigs with IDBA as the first stage of the MetaRon operon-prediction pipeline, producing scaffolded contigs that downstream gene, promoter, and operon prediction operate on.

## When to use

Use when starting from raw paired-end Illumina FASTQ shotgun reads (simulated via NeSSM or real, e.g. SRP029211) and scaftigs are needed as input for gene/operon prediction without a reference genome.

## When NOT to use

- Scaffolds and gene predictions are already available — use MetaRon's 'op' mode directly on scaftigs and the gene .tab file instead of reassembling
- Input is not shotgun reads but long/reference-quality genomes where reference-based operon annotation (e.g. DOOR/ODB/RegulonDB) is available
- Reads are not filtered or are single-end in an unsupported layout, violating MetaRon's paired-end FASTQ input requirements

## Inputs

- Filtered paired-end Illumina shotgun FASTQ reads (e.g. two million NeSSM-simulated reads at 15X coverage, or SRA reads such as SRP029211)
- Read layout flag: 'merge' (paired-end in two files) or 'paired' (paired-end in one file)
- Read length flag: 'l' (longer than 128 bases) or 'r' (shorter than 128 bases)

## Outputs

- Scaftigs from IDBA assembly (e.g. ERR022075_scaf.fa), with scaftigs <= 500 bp removed
- Downstream MetaRon outputs built on the scaftigs: Prodigal gene predictions (.tab), promoter predictions via NNPP 2.2, and the canonical Operon_File.tab

## How to apply

Run MetaRon in 'ago' mode (metaron --process ago) on paired-end FASTQ files, which invokes IDBA (iterative De Bruijn Graph de novo assembler) to assemble reads into scaftigs; in the E. coli K-12 MG1655 HiSeq run this yielded 82 scaftigs. Read type must be specified as 'merge' (reads in two files) or 'paired' (reads in one file), and read length as 'l' (>128 bases) or 'r' (<128 bases). Filter out scaftigs <= 500 bp before proceeding, because short scaftigs are discarded downstream. The resulting scaftig file (e.g. ERR022075_scaf.fa) then feeds Prodigal gene prediction, NNPP 2.2 promoter prediction, and proximon/operon identification with the IGD < 601 bp co-directionality rule. The rationale is that operon prediction in metagenomes cannot rely on reference genomes, so contiguity from de novo assembly is the substrate for gene-context inference.

## Related tools

- **MetaRon** (Command-line pipeline wrapper that orchestrates the assembly ('ago' mode) and passes scaftigs to gene and operon prediction) — github.com/zaidissa/MetaRon
- **IDBA** (Iterative De Bruijn Graph de novo assembler that builds scaftigs from the paired-end shotgun reads)
- **NeSSM** (Simulator used to generate the two million paired-end Illumina reads at 15X depth from the three reference genomes for the simulated metagenome)

## Examples

```
metaron --sample ERR022075 --process ago --read_type merge --read_length r --paired_1 ~/path/to/ERR022075.1.fastq --paired_2 ~/path/to/ERR022075.2.fastq --output ~/path/to/output/directory/
```

## Evaluation signals

- Scaftig file is produced in FASTA format (e.g. ERR022075_scaf.fa) and all scaftigs are longer than 500 bp after filtering
- Assembly scale is consistent with reported runs (e.g. 82 scaftigs and 4227 Prodigal-predicted genes for the E. coli K-12 MG1655 HiSeq dataset)
- Downstream output Operon_File.tab is generated with operon calls consistent with the IGD < 601 bp co-directional proximon rule
- On the simulated three-genome mixture, downstream performance should reach sensitivity ~93.7%, specificity ~75.5%, accuracy ~88.1% against DOOR reference operons
- The metaron command completes only when --sample, --process, --read_type, --read_length, --paired_1, --paired_2, and --output are all supplied

## Limitations

- MetaRon requires Python 2.7 (article reports Python > 3.0) and is restricted to Linux and academic use only
- The NNPP 2.2 promoter-prediction step can fail to identify promoters, limiting downstream operon confirmation
- Secondary metabolite prediction, pathway annotation, and graphical representation are not integrated into the pipeline (planned future work); BOWTIE/antiSMASH are optional downstream extras

## Evidence

- [results] 82 scaftigs were assembled by MetaRon via IDBA [55].: "82 scaftigs were assembled by MetaRon via IDBA [55]."
- [results] Scaftigs with length less than or equal to 500 bp were removed.: "Scaftigs with length less than or equal to 500 bp were removed."
- [readme] assembly into scaffolds via IDBA, data manipulation, gene prediction via prodigal and lastly operon prediction: "It comes with options to process the metagenomic data starting from filtered raw reads, which includes: assembly into scaffolds via IDBA, data manipulation, gene prediction via prodigal and lastly"
- [results] two million reads simulated at 15X depth via NeSSM: "two million reads simulated at 15X depth via NeSSM (Next-Generation Simulator for Metagenomics) [66]."
