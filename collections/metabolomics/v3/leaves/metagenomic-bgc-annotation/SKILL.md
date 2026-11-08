---
name: metagenomic-bgc-annotation
description: Use when you have assembled genomes or metagenomes (e.g., 40 cyanobacterial
  samples from the PoDP) and need annotated BGC GenBank files plus a BGC-BGC similarity
  matrix to link metabolites to gene clusters.
when_to_use_negative:
- Input is MS/MS spectral data only with no genome or metagenome to mine for BGCs
  — the annotation step requires assembled nucleotide sequence
- The target organism has no BGCs similar to the training set (e.g., a fungal species
  dissimilar to the PoDP-derived training BGCs), so downstream fingerprint matching
  yields no answer
- The goal is de novo metabolite structure elucidation rather than BGC-metabolite
  linking — NPOmix-style correlation cannot confirm 2D/3D structures
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3837
- http://edamontology.org/topic_3370
- http://edamontology.org/topic_0080
tools:
- name: antiSMASH
  role: Annotates biosynthetic gene clusters in each cyanobacterial genome/metagenome
    and emits the GenBank files consumed by BiG-SCAPE
- name: BiG-SCAPE
  role: Computes pairwise similarity (1 minus raw distance) between all training and
    sample BGCs using the Pfam-A.hmm domain definitions, grouping them into gene cluster
    families (GCFs)
  repo: https://bigscape-corason.secondarymetabolites.org
- name: NPOmix
  role: Downstream KNN classifier that consumes the BGC fingerprints derived from
    the concatenated BiG-SCAPE similarity table
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: MIBiG database
  role: Expert-annotated reference BGCs included in the BiG-SCAPE network via the
    --mibig flag to anchor GCF labels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:a90e813d047459ef1e58af4afaabdea76fead6d3d58f43d46dd24f75afc591b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - MIBiG database
  - NPOmix
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3837
  - http://edamontology.org/topic_3370
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# metagenomic-bgc-annotation

## Summary

Annotates biosynthetic gene clusters (BGCs) in cyanobacterial genomes and metagenomes with antiSMASH and computes pairwise BGC similarity networks with BiG-SCAPE, producing the similarity tables and fingerprints that downstream NPOmix-style KNN classifiers consume.

## When to use

Use when you have assembled genomes or metagenomes (e.g., 40 cyanobacterial samples from the PoDP) and need annotated BGC GenBank files plus a BGC-BGC similarity matrix to link metabolites to gene clusters.

## When NOT to use

- Input is MS/MS spectral data only with no genome or metagenome to mine for BGCs — the annotation step requires assembled nucleotide sequence
- The target organism has no BGCs similar to the training set (e.g., a fungal species dissimilar to the PoDP-derived training BGCs), so downstream fingerprint matching yields no answer
- The goal is de novo metabolite structure elucidation rather than BGC-metabolite linking — NPOmix-style correlation cannot confirm 2D/3D structures

## Inputs

- Assembled genome or metagenome/MAG contig FASTA files (e.g., 40 cyanobacterial (meta)genomes from the Paired Omics Data Platform)
- Training-set antiSMASH BGC GenBank files (antismash_only_gbk.zip from Zenodo record 6637083)
- Pfam-A.hmm database (Pfam 35.0), hmmpress-pressed for BiG-SCAPE

## Outputs

- antiSMASH GenBank (.gbk) BGC annotation files placed in the antismash_gbk_only folder
- BiG-SCAPE per-class network output directories (e.g., hybrids_glocal) with pairwise BGC similarity/distances
- Concatenated BGC similarity table across all classes
- BGC fingerprint matrix (rows = BGCs, columns = genomes, values = maximum similarity scores with <0.7 treated as absent), optionally with a GCF label column for KNN training

## How to apply

Run antiSMASH (installed via 'conda install -c bioconda antismash') on each genome or metagenome-assembled genome and place the resulting GenBank files in the 'antismash_gbk_only' folder. Download and press the Pfam-A.hmm database (Pfam 35.0: wget, gunzip, hmmpress) so BiG-SCAPE can domain-annotate the BGCs. Run BiG-SCAPE over all BGCs (training set from Zenodo record 6637083 plus your samples) with flags '--include_singletons' and '--mibig', which keeps singleton clusters and MIBiG reference BGCs in the network so query BGCs can be scored against validated GCFs. Concatenate the per-class network outputs (e.g., the hybrids_glocal files) into a single similarity table using 'sh concat_bigscape.sh'. Convert similarity scores (1 minus BiG-SCAPE raw distance, range 0.0–1.0) into BGC fingerprints by taking the maximum similarity per genome and treating scores below the 0.7 similarity cutoff as absent.

## Related tools

- **antiSMASH** (Annotates biosynthetic gene clusters in each cyanobacterial genome/metagenome and emits the GenBank files consumed by BiG-SCAPE)
- **BiG-SCAPE** (Computes pairwise similarity (1 minus raw distance) between all training and sample BGCs using the Pfam-A.hmm domain definitions, grouping them into gene cluster families (GCFs)) — https://bigscape-corason.secondarymetabolites.org
- **NPOmix** (Downstream KNN classifier that consumes the BGC fingerprints derived from the concatenated BiG-SCAPE similarity table) — https://github.com/tiagolbiotech/NPOmix_python
- **MIBiG database** (Expert-annotated reference BGCs included in the BiG-SCAPE network via the --mibig flag to anchor GCF labels)

## Examples

```
conda activate bigscape; bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig
```

## Evaluation signals

- Every input genome yields an antiSMASH .gbk file present in the antismash_gbk_only folder before BiG-SCAPE is invoked
- BiG-SCAPE was run against the hmmpress-pressed Pfam 35.0 Pfam-A.hmm with --include_singletons and --mibig; singletons and MIBiG BGCs appear in the network files
- All per-class network outputs (e.g., hybrids_glocal) are concatenated into one similarity table; row/column counts match the total BGC count (training + samples)
- BGC fingerprint values are similarity scores in [0.0, 1.0]; scores below the 0.7 cutoff are recoded as absent (0.0), and identical BGCs score 1.0
- The training matrix carries one extra column labeling each BGC fingerprint with its GCF so the KNN algorithm knows which fingerprint patterns belong together

## Limitations

- Metagenome quality can hinder BGC assembly, yielding fewer BGCs than expected and many fragmented BGCs
- Complete genomes are preferred; metagenomes and MAGs are supported but assembly fragmentation degrades the fingerprints
- The fingerprinting approach is correlation-based and cannot distinguish isomers or regioselectivity of homologous BGCs

## Evidence

- [other] workflow antiSMASH step: [evidence span withheld — anchor status: not_found]
- [readme] BiG-SCAPE invocation: "bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ \ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig"
- [methods] Pfam database download: "wget http://ftp.ebi.ac.uk/pub/databases/Pfam/releases/Pfam35.0/Pfam-A.hmm.gz"
- [readme] similarity cutoff semantics: "a score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
- [discussion] metagenome fragmentation caveat: "the quality of their metagenomes can hinder BGC assembly, leading to a smaller number of BGCs than expected and many of these BGCs can be fragmented"
