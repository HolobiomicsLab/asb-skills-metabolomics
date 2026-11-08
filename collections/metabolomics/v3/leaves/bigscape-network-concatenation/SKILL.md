---
name: bigscape-network-concatenation
description: Use when a set of antiSMASH GenBank BGCs (training set plus query genomes)
  must be converted into pairwise similarity scores and GCF assignments feeding an
  NPOmix BGC fingerprint matrix.
when_to_use_negative:
- You need MS/MS-to-MS/MS similarities (GNPS modified cosine scores), not BGC-to-BGC
  comparisons — that fingerprinting step is separate in NPOmix.
- Query BGCs have no homolog in the NPOmix training set (e.g. fungal BGCs dissimilar
  to the PoDP-derived bacterial training data); the downstream output would contain
  no answer.
- You only need antiSMASH annotation/prediction without cross-genome BGC similarity
  networking.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_3174
tools:
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity (1 minus raw distance) and groups BGCs into
    gene cluster families (GCFs) whose per-class outputs are concatenated for NPOmix
- name: antiSMASH
  role: Produces the GenBank BGC annotations for training and query genomes that BiG-SCAPE
    consumes as input
- name: NPOmix
  role: Downstream KNN classifier that consumes the concatenated BiG-SCAPE network
    to build BGC fingerprints binned at the 0.7 similarity cutoff
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: MIBiG
  role: Known BGC reference set included in the BiG-SCAPE run via --mibig so query
    BGCs are scored against validated clusters
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:8ccccd5d5ded502dea213d3b2e2edd0f7ec1e0f9f2d0c917010c52a189795bf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - MIBiG
  - MIBiG database
  - NPOmix
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# bigscape-network-concatenation

## Summary

Run BiG-SCAPE pairwise BGC similarity comparisons on antiSMASH-predicted BGCs (plus MIBiG and singletons) and concatenate the per-class output networks (e.g. hybrids_glocal) into the unified similarity network that NPOmix uses to build BGC fingerprints.

## When to use

Use when a set of antiSMASH GenBank BGCs (training set plus query genomes) must be converted into pairwise similarity scores and GCF assignments feeding an NPOmix BGC fingerprint matrix.

## When NOT to use

- You need MS/MS-to-MS/MS similarities (GNPS modified cosine scores), not BGC-to-BGC comparisons — that fingerprinting step is separate in NPOmix.
- Query BGCs have no homolog in the NPOmix training set (e.g. fungal BGCs dissimilar to the PoDP-derived bacterial training data); the downstream output would contain no answer.
- You only need antiSMASH annotation/prediction without cross-genome BGC similarity networking.

## Inputs

- antiSMASH GenBank output folder (antismash_gbk_only) for query samples plus the training BGC set (antismash_only_gbk.zip, Zenodo record 6637083)
- Pfam-A.hmm from Pfam 35.0 (hmmpress-ed)
- BiG-SCAPE conda environment and the concat_bigscape.sh helper script from the NPOmix_python repository

## Outputs

- BiG-SCAPE per-class network output directories (raw distance and similarity scores, singleton clustering)
- Concatenated cross-class BGC similarity network (e.g. 2023-11-11_11-11-11_hybrids_glocal/) used to build NPOmix BGC fingerprints and GCF labels

## How to apply

Install BiG-SCAPE in a dedicated conda environment (`conda create -n bigscape -c bioconda bigscape`), download and hmmpress Pfam-A.hmm from Pfam 35.0, and run BiG-SCAPE on the antiSMASH GBK folder with `--pfam_dir`, `-c 8`, `--include_singletons`, and `--mibig` so that query BGCs are compared against the MIBiG-containing training set (e.g. 5,421 BGCs across 1,040 genomes/metagenomes from Zenodo record 6637083). BiG-SCAPE emits separate network files per BGC class (PKS, NRPS, RiPPs, saccharides, hybrids, etc.), so run the `concat_bigscape.sh` script over the output directory to merge all classes into a single glocal network (e.g. `2023-11-11_11-11-11_hybrids_glocal/`). Similarity scores are computed as 1 minus BiG-SCAPE raw distance, ranging 0.0–1.0, and are later binarized at the 0.7 similarity cutoff when building NPOmix fingerprints. The concatenated network is the input NPOmix uses to take, per query BGC, the maximum similarity to any BGC in each genome, producing the training matrix whose rows are labeled with GCFs.

## Related tools

- **BiG-SCAPE** (Computes pairwise BGC similarity (1 minus raw distance) and groups BGCs into gene cluster families (GCFs) whose per-class outputs are concatenated for NPOmix)
- **antiSMASH** (Produces the GenBank BGC annotations for training and query genomes that BiG-SCAPE consumes as input)
- **NPOmix** (Downstream KNN classifier that consumes the concatenated BiG-SCAPE network to build BGC fingerprints binned at the 0.7 similarity cutoff) — https://github.com/tiagolbiotech/NPOmix_python
- **MIBiG** (Known BGC reference set included in the BiG-SCAPE run via --mibig so query BGCs are scored against validated clusters)

## Examples

```
bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig && sh concat_bigscape.sh /path/to/bigscape_output/ 2023-11-11_11-11-11_hybrids_glocal/
```

## Evaluation signals

- BiG-SCAPE completes on 8 cores without Pfam errors (Pfam-A.hmm downloaded from Pfam 35.0, hmmpress-ed, passed via --pfam_dir).
- Per-class output directories exist for all BGC classes (including hybrids) and --include_singletons retains singleton BGCs in the network.
- concat_bigscape.sh produces a single merged folder named with the hybrids_glocal convention (e.g. 2023-11-11_11-11-11_hybrids_glocal/).
- Downstream NPOmix fingerprinting yields similarity scores in 0.0–1.0 range with scores below the 0.7 cutoff binarized to absent.
- Reproducing the published setup recovers the GCF labels that give NPOmix top-1 = 81.8% and top-10 = 92.9% precision on GNPS reference spectra.

## Limitations

- Metagenome quality can hinder BGC assembly, yielding fewer and fragmented BGCs that weaken the similarity network.
- The per-class BiG-SCAPE outputs must be concatenated manually (concat_bigscape.sh) — class-split outputs alone do not form the complete NPOmix network.
- Similarity scores are relative to the training set; a query species with no similar BGC in the PoDP-derived training data produces no usable network signal.

## Evidence

- [readme] BiG-SCAPE run command with parameters: "bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig"
- [readme] Concatenation step into glocal network: "sh concat_bigscape.sh /path/to/bigscape_output/ 2023-11-11_11-11-11_hybrids_glocal/"
- [readme] Pairwise comparison rationale: "each biosynthetic gene cluster (BGC) in the genomes will go through a pairwise similarity comparison (Fig. 1A) to every other BGC in the same set of genomes to compute similarity scores (1 minus"
- [readme] Score scale and cutoff semantics: "Similarity scores range from 0.0 to 1.0; for instance, identical BGCs have a perfect similarity score of 1.0, a score of 0.8 would represent that a homologous BGC is present in the genome"
- [methods] Pfam database setup prerequisite: "wget http://ftp.ebi.ac.uk/pub/databases/Pfam/releases/Pfam35.0/Pfam-A.hmm.gz"
