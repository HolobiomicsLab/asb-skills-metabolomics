---
name: paired-omics-data-retrieval
description: Use when a linking or benchmarking analysis needs paired genome+LC-MS/MS
  samples with validated BGC–metabolite links, e.g. from the PoDP or a GNPS job task,
  to build fingerprints or evaluate precision/recall.
when_to_use_negative:
- The analysis needs only single-omics data (genomes alone or spectra alone) with
  no pairing requirement — plain genome mining or GNPS library search suffices
- The target organism group lacks paired data in PoDP (e.g. plants, or RiPP-class
  validated metabolites), so no usable paired validation set can be assembled
- The goal is MS/MS library dereplication rather than linking spectra to biosynthetic
  gene clusters
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3365
tools:
- name: Paired Omics Data Platform (PoDP)
  role: Primary source of paired genome+metabolome samples with validated BGC–mass
    spectral links used for training and validation
  repo: https://pairedomicsdata.bioinformatics.nl
- name: GNPS
  role: Hosts the LC-MS/MS metabolome job; cloned and extended to supply spectra for
    the paired samples
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: antiSMASH
  role: Generates the BGC GenBank annotations that constitute the genomic half of
    each paired sample
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:af0b0d8b2016190b9ebfe82b78f01ed670d6f898fd7be88372bd4277fc3bdfca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - NPOmix
  - NRPminer
  - Paired Omics Data Platform (PoDP)
  - antiSMASH
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# paired-omics-data-retrieval

## Summary

Retrieves paired genome–metabolome sample sets (genomes/metagenomes plus matching untargeted LC-MS/MS data) from the Paired Omics Data Platform and GNPS, the prerequisite input layer for NPOmix-style BGC–MS/MS linking and benchmarking.

## When to use

Use when a linking or benchmarking analysis needs paired genome+LC-MS/MS samples with validated BGC–metabolite links, e.g. from the PoDP or a GNPS job task, to build fingerprints or evaluate precision/recall.

## When NOT to use

- The analysis needs only single-omics data (genomes alone or spectra alone) with no pairing requirement — plain genome mining or GNPS library search suffices
- The target organism group lacks paired data in PoDP (e.g. plants, or RiPP-class validated metabolites), so no usable paired validation set can be assembled
- The goal is MS/MS library dereplication rather than linking spectra to biosynthetic gene clusters

## Inputs

- Paired Omics Data Platform (PoDP) records with paired genome/metagenome and metabolome metadata for the same samples
- GNPS job task ID (e.g. b915d1f78c364bf4be9900047760d95c) hosting the LC-MS/MS metabolomes
- Zenodo archive of antiSMASH BGC GenBank files (antismash_only_gbk.zip, record 6637083)

## Outputs

- Curated set of paired genome–MS/MS validation samples (9 samples / 22 validated BGC–metabolite links in the reference workflow)
- Cloned GNPS job containing the metabolome files arranged in analysis groups (e.g. G3)
- antiSMASH BGC .gbk files placed in the antismash_gbk_only folder for BiG-SCAPE/NPOmix ingestion

## How to apply

Identify the paired samples (each with both a genome or metagenome and MS/MS spectra from untargeted LC-MS/MS) in the Paired Omics Data Platform (PoDP), filtering to samples with experimentally validated BGC–metabolite links when assembling a validation set (NPOmix used 9 paired samples covering 22 validated links). Retrieve the metabolome side via the GNPS job (e.g. task=b915d1f78c364bf4be9900047760d95c), cloning the job and adding your own files to the designated group (G3 in the README workflow). Retrieve the genome side as antiSMASH BGC annotations — download the training/validated BGC GenBank archive from Zenodo record 6637083 (antismash_only_gbk.zip) — and record any tools or samples that cannot be retrieved. Verify that every retrieved sample has both halves of the pair and that the number of validated links matches the reference set before downstream KNN fingerprinting or benchmark scoring.

## Related tools

- **Paired Omics Data Platform (PoDP)** (Primary source of paired genome+metabolome samples with validated BGC–mass spectral links used for training and validation) — https://pairedomicsdata.bioinformatics.nl
- **GNPS** (Hosts the LC-MS/MS metabolome job; cloned and extended to supply spectra for the paired samples) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **antiSMASH** (Generates the BGC GenBank annotations that constitute the genomic half of each paired sample)

## Examples

```
wget -O antismash_bgcs.zip https://zenodo.org/record/6637083/files/antismash_only_gbk.zip?download=1 && unzip antismash_bgcs.zip && rm antismash_bgcs.zip
```

## Evaluation signals

- Every retrieved sample contains BOTH a genome/metagenome and LC-MS/MS spectra — unpaired samples must be dropped or flagged
- The validation subset matches the reference count: 9 paired samples covering the 22 experimentally validated BGC–metabolite links
- The GNPS clone is rooted at task=b915d1f78c364bf4be9900047760d95c and uploaded user files are placed in the G3 group as the README instructs
- antiSMASH GenBank files are present in the antismash_gbk_only folder and their BGC count is consistent with the PoDP sample metadata
- Any tool/sample that could not be retrieved or executed is explicitly recorded (as the benchmark does in Table S3) rather than silently omitted

## Limitations

- There is a current lack of paired data for plants; PoDP paired data used by NPOmix is almost exclusively from bacteria
- No RiPP-class validated metabolites exist in the PoDP training data, so the retrieved validation set cannot cover that BGC class
- Metagenome quality can hinder BGC assembly, yielding fewer and fragmented BGCs even when paired data are available

## Evidence

- [abstract] Using paired datasets that include validated genes-mass spectral links from the Paired Omics Data Platform: "Using paired datasets that include validated genes-mass spectral links from the Paired Omics Data Platform"
- [intro] Paired Omics Data Platform (PoDP) (20), which contains paired data (genome and metabolome data of the same samples): "Paired Omics Data Platform (PoDP) (20), which contains paired data (genome and metabolome data of the same samples)"
- [readme] Clone the GNPS job with the metabolomes for the strains in the training set and add your files (ideally to G3 group): "Clone the GNPS job with the metabolomes for the strains in the training set and add your files (ideally to G3 group)"
- [readme] Many paired datasets are available at the Paired omics Data Platform, one of the first initiatives to gather paired genome-MS/MS samples: "Many paired datasets are available at the Paired omics Data Platform [(PoDP)](https://pairedomicsdata.bioinformatics.nl), one of the first initiatives to gather paired genome-MS/MS samples."
- [discussion] the quality of their metagenomes can hinder BGC assembly, leading to a smaller number of BGCs than expected: "the quality of their metagenomes can hinder BGC assembly, leading to a smaller number of BGCs than expected and many of these BGCs can be fragmented"
