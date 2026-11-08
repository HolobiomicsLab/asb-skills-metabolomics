---
name: paired-omics-dataset-curation
description: Use when building a genome-mining workflow that needs matched genome
  (or metagenome) and metabolome data for the same samples, e.g., to train or query
  NPOmix against PoDP paired datasets.
when_to_use_negative:
- The samples lack a genome (or metagenome) paired with MS/MS spectra of the same
  organism — NPOmix needs both sides of each pair
- The query organism has no BGC similar to any training-set BGC (e.g., a fungal species
  with no similar BGCs), which yields no answer
- You need experimentally confirmed metabolite structures rather than computational
  links — new NPOmix predictions still require proper elucidation of planar (2D) and
  absolute (3D) structure
edam_operation: http://edamontology.org/operation_3760
edam_topics:
- http://edamontology.org/topic_3407
- http://edamontology.org/topic_3174
tools:
- name: NPOmix
  role: Consumes the curated paired dataset as a labeled training matrix for its k-nearest
    neighbor classifier
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: antiSMASH
  role: Annotates biosynthetic gene clusters in each curated genome; GBK outputs are
    placed in the antismash_gbk_only folder
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity (with Pfam-A.hmm, --include-singletons, --mibig)
    used to build BGC fingerprints over the curated genomes
- name: GNPS
  role: Computes pairwise modified cosine scores between query MS/MS spectra and experimental
    spectra of every curated sample
- name: Paired Omics Data Platform (PoDP)
  role: Source of paired genome–metabolome datasets used for curation and training
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:d3364f9df7be45676fb556e0110da8362c179f59705f90702549498225b82edf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GNPS
  - NPOmix
  - Paired Omics Data Platform (PoDP)
  - antiSMASH
  edam_operation: http://edamontology.org/operation_3760
  edam_topics:
  - http://edamontology.org/topic_3407
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# paired-omics-dataset-curation

## Summary

Curation and assembly of paired genome–MS/MS samples (each containing a genome or metagenome plus untargeted LC-MS/MS spectra of the same organism), retrieved from the Paired Omics Data Platform (PoDP), to serve as the labeled training matrix for NPOmix's k-nearest neighbor classifier linking metabolites to biosynthetic gene clusters.

## When to use

Use when building a genome-mining workflow that needs matched genome (or metagenome) and metabolome data for the same samples, e.g., to train or query NPOmix against PoDP paired datasets.

## When NOT to use

- The samples lack a genome (or metagenome) paired with MS/MS spectra of the same organism — NPOmix needs both sides of each pair
- The query organism has no BGC similar to any training-set BGC (e.g., a fungal species with no similar BGCs), which yields no answer
- You need experimentally confirmed metabolite structures rather than computational links — new NPOmix predictions still require proper elucidation of planar (2D) and absolute (3D) structure

## Inputs

- Paired genome–MS/MS sample records from the Paired Omics Data Platform (PoDP), e.g., the 40 cyanobacterial (meta)genomes including Moorena producens PAL/JHB
- Training antiSMASH BGC GenBank files (antismash_only_gbk.zip from Zenodo record 6637083)
- antiSMASH-annotated BGC GenBank files for each curated sample genome
- Experimental LC-MS/MS spectra (multiple files per genome merged into one spectral set)
- GNPS reference spectra and MIBiG entries with experimentally validated gene–metabolite links

## Outputs

- Curated set of paired genome–MS/MS samples suitable for NPOmix training
- BGC training matrix: rows of maximum BiG-SCAPE similarity scores per genome, each row labeled with a GCF
- MS/MS testing matrix of maximum modified cosine similarity scores per sample
- Metabolite–BGC link predictions (npomix_predictions.csv)

## How to apply

Retrieve paired genome–MS/MS samples from the Paired Omics Data Platform (PoDP), which stores genome and metabolome data of the same samples; for the NPOmix cyanobacterial case this includes paired data for 40 cyanobacterial (meta)genomes such as Moorena producens PAL/JHB. Annotate BGCs in each genome with antiSMASH and place the resulting GenBank files in the antismash_gbk_only folder, then include the training antiSMASH BGC GenBank files (antismash_only_gbk.zip from Zenodo record 6637083) alongside sample BGCs. Process every sample's experimental MS/MS spectra together with reference spectra from GNPS so that pairwise modified cosine scores can later be computed between each query spectrum and the spectra of every sample. Multiple LC-MS/MS files for the same genome are merged into a single set of experimental MS/MS spectra, and complete genomes are preferred over fragments because metagenome quality can fragment BGCs. The curated pairs ultimately form a training matrix in which rows are BGC fingerprints (maximum similarity per genome, scores below the 0.7 BiG-SCAPE cutoff treated as absent) labeled with a GCF for the KNN algorithm.

## Related tools

- **NPOmix** (Consumes the curated paired dataset as a labeled training matrix for its k-nearest neighbor classifier) — https://github.com/tiagolbiotech/NPOmix_python
- **antiSMASH** (Annotates biosynthetic gene clusters in each curated genome; GBK outputs are placed in the antismash_gbk_only folder)
- **BiG-SCAPE** (Computes pairwise BGC similarity (with Pfam-A.hmm, --include-singletons, --mibig) used to build BGC fingerprints over the curated genomes)
- **GNPS** (Computes pairwise modified cosine scores between query MS/MS spectra and experimental spectra of every curated sample)
- **Paired Omics Data Platform (PoDP)** (Source of paired genome–metabolome datasets used for curation and training)

## Examples

```
wget -O antismash_bgcs.zip https://zenodo.org/record/6637083/files/antismash_only_gbk.zip?download=1; unzip antismash_bgcs.zip && rm antismash_bgcs.zip
```

## Evaluation signals

- Each curated sample has both a genome/metagenome (with antiSMASH GBK output in antismash_gbk_only) and a merged set of experimental MS/MS spectra
- BGC fingerprints use maximum similarity per genome with scores below the 0.7 BiG-SCAPE cutoff treated as absent (similarity = 1 minus BiG-SCAPE raw distance)
- Training matrix contains a GCF label column per BGC fingerprint row (e.g., round 4 used 5,421 BGCs across 1,040 genomes/metagenomes)
- Known metabolite–BGC links are reproduced: 18 known spectra correctly linked to validated genes, including palmyramide A when substructure features were included
- Precision of predictions should approximate published values (top-1 = 81.8%, top-10 = 92.9% with Jaccard co-occurrence threshold)

## Limitations

- Metagenome quality can hinder BGC assembly, yielding fewer and fragmented BGCs than expected
- No RiPP-class validated metabolites currently exist in the PoDP training data, and paired data for plants is lacking (demonstration data is almost exclusively bacterial)
- NPOmix cannot distinguish isomers or regioselectivity of homologs, and precision may not be reproducible for datasets dissimilar to the training set
- There is a large disparity between known metabolites from an organism and BGCs with known metabolite products (e.g., Moorena has 200+ metabolites but only ~a dozen validated BGCs in MIBiG)

## Evidence

- [intro] PoDP paired dataset source: "Paired Omics Data Platform (PoDP) (20), which contains paired data (genome and metabolome data of the same samples)"
- [readme] Merging multiple MS/MS files per genome: "This KNN approach also supports LC-MS/MS from fractions or from different culture conditions; multiple LC-MS/MS files for the same genome were merged together into a single set of experimental MS/MS"
- [readme] Genome preference rationale: "Metagenomes and metagenome-assembled genomes can also be used instead of genomes; however, complete genomes are preferred."
- [readme] Training matrix construction and GCF labels: "one extra column is required in this genomic data matrix, a column that labels each BGC fingerprint with a GCF so the KNN algorithm will know the fingerprint patterns that belong together"
- [methods] Zenodo training BGC download command: "wget -O antismash_bgcs.zip https://zenodo.org/record/6637083/files/antismash_only_gbk.zip?download=1"
- [discussion] Metagenome quality limitation: "the quality of their metagenomes can hinder BGC assembly, leading to a smaller number of BGCs than expected and many of these BGCs can be fragmented"
