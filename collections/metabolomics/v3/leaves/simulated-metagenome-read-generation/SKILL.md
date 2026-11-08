---
name: simulated-metagenome-read-generation
description: Use when a benchmark requires realistic paired-end shotgun reads from
  known reference genomes so pipeline outputs (e.g., operons) can be scored against
  curated per-genome truth annotations.
when_to_use_negative:
- Real metagenomic samples are the input and the goal is biological discovery rather
  than controlled benchmarking against known truth
- The target genomes lack curated operon annotations (e.g., DOOR), so simulated reads
  cannot be scored against ground truth
- Pre-assembled scaffolds and gene predictions are already available, in which case
  MetaRon's 'op' mode operates directly on scaftigs and gene predictions without simulated
  reads
edam_operation: http://edamontology.org/operation_3559
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: NeSSM
  role: Simulates two million Illumina reads at 15X depth from the three reference
    genomes to create the metagenome mixture used for benchmarking MetaRon
- name: MetaRon
  role: Consumer of the simulated reads in 'ago' mode (metaron --process ago), which
    assembles, predicts genes, predicts promoters, and calls operons that are then
    scored against DOOR
  repo: https://github.com/zaidissa/MetaRon
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12864-020-07357-5
    title: 'Prediction and analysis of metagenomic operons via MetaRon: a pipeline
      for prediction of Metagenome and whole-genome opeRons'
  source_content_hash: sha256:1ac228221c9718528cb98ba880da8b3baff247ba7f77b88bb3c949bc99010792
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zaidissa/MetaRon
  tools:
  - IDBA
  - MetaRon
  - NNPP
  - NeSSM
  edam_operation: http://edamontology.org/operation_3559
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# simulated-metagenome-read-generation

## Summary

Generate in silico shotgun metagenome reads from reference genomes using NeSSM so that operon prediction pipelines like MetaRon can be benchmarked against curated ground-truth annotations (e.g., DOOR). In the source work, two million Illumina reads at 15X depth were simulated from a three-genome mixture of E. coli MG1655, M. tuberculosis H37Rv and B. subtilis 168.

## When to use

Use when a benchmark requires realistic paired-end shotgun reads from known reference genomes so pipeline outputs (e.g., operons) can be scored against curated per-genome truth annotations.

## When NOT to use

- Real metagenomic samples are the input and the goal is biological discovery rather than controlled benchmarking against known truth
- The target genomes lack curated operon annotations (e.g., DOOR), so simulated reads cannot be scored against ground truth
- Pre-assembled scaffolds and gene predictions are already available, in which case MetaRon's 'op' mode operates directly on scaftigs and gene predictions without simulated reads

## Inputs

- Reference genome sequences/Accessions: E. coli MG1655 NC_000913.3, M. tuberculosis H37Rv NC_000962, B. subtilis 168 NC_000964
- Simulation design parameters: number of reads (two million), coverage (15X), Illumina read model
- Curated reference operon annotations (e.g., ODB/DOOR/RegulonDB) for ground-truth evaluation

## Outputs

- Simulated paired-end Illumina FASTQ files representing a three-genome metagenome mixture
- Downstream benchmark table (e.g., simulated_genome_operon_performance.csv) with sensitivity 93.7%, specificity 75.5%, accuracy 88.1%

## How to apply

Simulate reads from the chosen reference genomes (here E. coli MG1655 NC_000913.3, M. tuberculosis H37Rv NC_000962 and B. subtilis 168 NC_000964) using NeSSM (Next-Generation Simulator for Metagenomics), specifying an Illumina read model, two million reads, and 15X coverage to mimic the source study. Combine the per-genome reads into one paired-end FASTQ mixture to emulate a metagenomic sample. The simulated paired-end FASTQ files then feed downstream processing (e.g., metaron --process ago with IDBA assembly, scaftig filtering <= 500 bp, Prodigal gene prediction and NNPP2.2 promoter prediction). Correctness is judged by confirming read counts/coverage match the design (2M reads, 15X) and that downstream operon metrics against DOOR (sensitivity 93.7%, specificity 75.5%, accuracy 88.1%) reproduce within tolerance (e.g., +/- 1 percentage point).

## Related tools

- **NeSSM** (Simulates two million Illumina reads at 15X depth from the three reference genomes to create the metagenome mixture used for benchmarking MetaRon)
- **MetaRon** (Consumer of the simulated reads in 'ago' mode (metaron --process ago), which assembles, predicts genes, predicts promoters, and calls operons that are then scored against DOOR) — https://github.com/zaidissa/MetaRon

## Examples

```
metaron --sample mix3 --process ago --read_type merge --read_length r --paired_1 mix3_1.fastq --paired_2 mix3_2.fastq --output ./out/
```

## Evaluation signals

- Simulated dataset contains the designed read count (two million reads) and coverage (15X) over the three-genome mixture
- Paired-end FASTQ outputs are well-formed and readable by the downstream pipeline (IDBA assembly succeeds; scaftigs <= 500 bp are removed)
- Downstream operon metrics against DOOR match the reported simulated-genome values: sensitivity 93.7%, specificity 75.5%, accuracy 88.1% within +/- 1 percentage point
- Performance table contains sensitivity, specificity and accuracy fields, e.g., saved as simulated_genome_operon_performance.csv
- Predicted operons (2579 operons, 8749 genes in the source run) are of the same order of magnitude as the reported result

## Limitations

- Results depend on the simulator's read model (NeSSM, Illumina HiSeq-style); other sequencing platforms or error profiles may shift downstream metrics
- Benchmark validity relies on the quality/completeness of curated truth operon annotations (DOOR/ODB/RegulonDB) for the chosen genomes
- Only three genomes were simulated, so performance on more complex communities or uneven abundance profiles is not established

## Evidence

- [results] two million reads simulated at 15X depth via NeSSM (Next-Generation Simulator for Metagenomics): "two million reads simulated at 15X depth via NeSSM (Next-Generation Simulator for Metagenomics) [66]."
- [abstract] simulated mixture of three whole-genomes: "simulated mixture of three whole-genomes (E. coli MG1655, Mycobacterium tuberculosis H37Rv and Bacillus subtilis str. 16)"
- [abstract] MetaRon achieved simulated-genome metrics: "simulated genome (93.7, 75.5 and 88.1%)"
- [results] Whole-genome accession for E. coli MG1655 used in simulation: "Whole-genome of Escherichia coli MG1655 (NC_000913.3)"
- [readme] metaron --process ago consumes the simulated reads: "metaron --sample ERR022075 --process ago --read_type merge OR paired --read_length r OR l --paired_1 ~/path/to/ERR022075.1.fastq --paired_2 ~/path/to/ERR022075.2.fastq --output"
