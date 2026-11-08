---
name: biosynthetic-gene-cluster-family-networking
description: Use when you have antiSMASH GenBank files for training-set and sample
  genomes and need a BGC-by-genome similarity matrix (scores 0–1, 0.7 cutoff) to feed
  a KNN classifier or define GCFs.
when_to_use_negative:
- The input genomes have no similar BGCs to the training set — NPOmix will produce
  no answer (e.g., a fungal species with no BGC similar to any PoDP training BGC).
- Metagenome quality is too low to assemble BGCs — fragmented metagenomes yield fewer,
  truncated BGCs and unreliable family assignments.
- The goal is distinguishing isomers or regioselectivity of homologous metabolites
  — this correlation-based networking lacks that resolution.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity scores (1 minus raw distance) and groups
    BGCs into gene cluster families (GCFs); per-class network outputs are concatenated
    into the similarity table.
  repo: https://bigscape-corason.secondarymetabolites.org
- name: antiSMASH
  role: Annotates BGCs in each genome/metagenome and produces the GenBank files that
    serve as the input to BiG-SCAPE.
- name: HMMER (hmmpress)
  role: Prepares the Pfam-A.hmm database (Pfam 35.0) used by BiG-SCAPE for domain
    detection.
- name: MIBiG
  role: Expert-annotated reference BGCs included in the network via the --mibig flag
    so sample BGCs can be compared against validated clusters.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:1482fc31e2b7f45e0510bccf4dd60250c8af555bee39784638706bff03a566d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - HMMER (hmmpress)
  - MIBiG
  - NPOmix
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# biosynthetic-gene-cluster-family-networking

## Summary

Computes pairwise similarities between antiSMASH-annotated biosynthetic gene clusters (BGCs) with BiG-SCAPE and assigns them to gene cluster families (GCFs), producing the BGC similarity matrix that NPOmix uses to build BGC fingerprints.

## When to use

Use when you have antiSMASH GenBank files for training-set and sample genomes and need a BGC-by-genome similarity matrix (scores 0–1, 0.7 cutoff) to feed a KNN classifier or define GCFs.

## When NOT to use

- The input genomes have no similar BGCs to the training set — NPOmix will produce no answer (e.g., a fungal species with no BGC similar to any PoDP training BGC).
- Metagenome quality is too low to assemble BGCs — fragmented metagenomes yield fewer, truncated BGCs and unreliable family assignments.
- The goal is distinguishing isomers or regioselectivity of homologous metabolites — this correlation-based networking lacks that resolution.

## Inputs

- antiSMASH GenBank (.gbk) files for training-set BGCs (antismash_only_gbk.zip from Zenodo record 6637083) and for sample genomes/metagenomes
- hmmpress-pressed Pfam-A.hmm database (Pfam 35.0)
- antiSMASH BGC GBK files for the user's own samples placed in the antismash_gbk_only folder

## Outputs

- Per-class BiG-SCAPE network output directories (e.g., hybrids_glocal files)
- A single concatenated BGC similarity table over all BGCs (training set plus samples)
- Gene cluster family (GCF) assignments used as labels for the KNN training matrix

## How to apply

First annotate BGCs in all genomes/metagenomes (e.g., the 40 cyanobacterial samples plus the training set) with antiSMASH and place the GBK files in the antismash_gbk_only folder. Download and parse Pfam-A.hmm (Pfam 35.0) and hmmpress it so BiG-SCAPE can detect domains. Run BiG-SCAPE over all BGCs — training set plus samples — with --include-singletons and --mibig so singletons are retained and MIBiG reference BGCs participate in the network. BiG-SCAPE emits per-class network outputs (e.g., hybrids_glocal files); concatenate these per-class outputs into a single similarity table where each value is the maximum similarity of a query BGC to any BGC in a given genome, computed as 1 minus the BiG-SCAPE raw distance. Treat similarity scores below the 0.7 cutoff as 0 (BGC absent from that genome); identical BGCs score 1.0 and homologs score around 0.8.

## Related tools

- **BiG-SCAPE** (Computes pairwise BGC similarity scores (1 minus raw distance) and groups BGCs into gene cluster families (GCFs); per-class network outputs are concatenated into the similarity table.) — https://bigscape-corason.secondarymetabolites.org
- **antiSMASH** (Annotates BGCs in each genome/metagenome and produces the GenBank files that serve as the input to BiG-SCAPE.)
- **HMMER (hmmpress)** (Prepares the Pfam-A.hmm database (Pfam 35.0) used by BiG-SCAPE for domain detection.)
- **MIBiG** (Expert-annotated reference BGCs included in the network via the --mibig flag so sample BGCs can be compared against validated clusters.)

## Examples

```
bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig
```

## Evaluation signals

- All BGCs — including singletons and MIBiG references — are present in the network because --include_singletons and --mibig were passed; no BGC silently dropped.
- Similarity scores fall in the 0.0–1.0 range: identical BGCs = 1.0, homologs ≈ 0.8, scores below the 0.7 cutoff treated as 0 (BGC absent).
- Concatenated similarity table contains rows for all per-class network outputs (e.g., hybrids_glocal) and one column per genome in the dataset (e.g., 1,040 genomes/metagenomes in the release).
- Each BGC fingerprint row in the downstream training matrix carries a GCF label column so the KNN algorithm knows which fingerprint patterns belong together.
- The Pfam database is hmmpress-pressed before running (Pfam-A.hmm from Pfam 35.0 release).

## Limitations

- Metagenome quality can hinder BGC assembly, leading to fewer BGCs than expected and many fragmented BGCs.
- The similarity-cutoff behavior (below 0.7 treated as absent) defines the fingerprint resolution; no finer distinction is retained.
- Results depend on having BGCs similar to the training set; genomes with no similar BGCs yield no network signal for those queries.

## Evidence

- [readme] bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ \ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig: "bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ \ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig"
- [readme] each biosynthetic gene cluster (BGC) in the genomes will go through a pairwise similarity comparison (Fig. 1A) to every other BGC in the same set of genomes to compute similarity scores (1 minus BiG-SCAPE raw distance) and to assign BGCs to Gene Cluster Families (GCFs): "each biosynthetic gene cluster (BGC) in the genomes will go through a pairwise similarity comparison (Fig. 1A) to every other BGC in the same set of genomes to compute similarity scores (1 minus"
- [readme] the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome: "the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
- [methods] Run BiG-SCAPE with the hmmpress-pressed Pfam-A.hmm database (Pfam 35.0), flags --include-singletons and --mibig, over all BGCs (training set plus samples), then concatenate the per-class network outputs (e.g., the hybrids_glocal files) into a single similarity table: [evidence span withheld — anchor status: not_found]
- [intro] BiG-SCAPE (7) can reliably calculate the similarity between pairs of BGCs, grouping them into gene cluster families (GCFs): "BiG-SCAPE (7) can reliably calculate the similarity between pairs of BGCs, grouping them into gene cluster families (GCFs)"
