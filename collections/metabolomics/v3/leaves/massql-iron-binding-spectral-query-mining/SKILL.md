---
name: massql-iron-binding-spectral-query-mining
description: Use when untargeted LC-MS/MS metabolome profiles from paired genome–metabolome
  samples (e.g., PoDP) must be screened for iron-bound siderophore-like spectra via
  a MassQL query, and the hits then linked to BGCs/GCFs.
when_to_use_negative:
- The organism of interest has no BGC similar to any BGC in the NPOmix training set
  (e.g., fungal species dissimilar to the PoDP-derived training data) — the classifier
  will return no answer
- The dataset differs substantially from the PoDP bacterial training set (e.g., plant
  paired data, which is currently lacking) — precision may not be reproducible
- The goal is resolving isomers or regioselectivity of homologous siderophores — this
  correlation-based approach lacks that resolution
tools:
- name: MassQL
  role: Query language used to filter iron-bound siderophore spectra from the pooled
    LC-MS/MS profiles before dereplication and classification
- name: NPOmix
  role: k-nearest neighbor classifier that projects MS/MS fingerprints into a BGC-fingerprint-trained
    space and votes GCF labels for each query spectrum
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: BiG-SCAPE
  role: Computes pairwise BGC similarities (1 − raw distance) and GCF assignments
    used to build the BGC fingerprint training matrix (0.7 similarity cutoff)
  repo: https://bigscape-corason.secondarymetabolites.org
- name: GNPS
  role: Provides pairwise modified cosine scoring between query and sample MS/MS spectra
    to build the MS/MS fingerprint matrix; also hosts the training metabolome job
  repo: https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- name: antiSMASH
  role: Produces the BGC annotations/GBK files (including siderophore-labeled BGCs
    such as Achromobacter xylosoxidans NH44784-1996) that constitute the training
    BGCs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:0d3254a1d1ff6971a34c99447b1ec3ca01ba00d86b16835f72144bdbc9c362e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GNPS
  - MassQL
  - NPOmix
  - antiSMASH
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# massql-iron-binding-spectral-query-mining

## Summary

A MassQL + NPOmix mining skill that filters iron-bound siderophore MS/MS spectra out of pooled untargeted LC-MS/MS profiles, then classifies the deduplicated query spectra against BiG-SCAPE BGC fingerprints with a k-nearest neighbor classifier to link them to siderophore gene cluster families.

## When to use

Use when untargeted LC-MS/MS metabolome profiles from paired genome–metabolome samples (e.g., PoDP) must be screened for iron-bound siderophore-like spectra via a MassQL query, and the hits then linked to BGCs/GCFs.

## When NOT to use

- The organism of interest has no BGC similar to any BGC in the NPOmix training set (e.g., fungal species dissimilar to the PoDP-derived training data) — the classifier will return no answer
- The dataset differs substantially from the PoDP bacterial training set (e.g., plant paired data, which is currently lacking) — precision may not be reproducible
- The goal is resolving isomers or regioselectivity of homologous siderophores — this correlation-based approach lacks that resolution

## Inputs

- Pooled LC-MS/MS metabolome files from paired genome–metabolome samples (e.g., Paired Omics Data Platform exports)
- MassQL iron-bound siderophore query specification
- antiSMASH GBK files for training and query genomes (e.g., antismash_only_gbk.zip from Zenodo record 6637083)
- BiG-SCAPE pairwise BGC similarity/network outputs (concatenated across classes, hybrids_glocal)
- GNPS job output for the training metabolomes (reference GNPS task b915d1f78c364bf4be9900047760d95c)

## Outputs

- Deduplicated, nonredundant MGF file of query MS/MS spectra (putative siderophore spectra)
- MS/MS similarity fingerprint matrix (query spectra rows × 1,040 paired-sample columns)
- GCF label votes per query spectrum from the trained kNN classifier
- Link table (e.g., siderophore_npomix_links.tsv) mapping each spectrum to predicted siderophore GCF(s) with Jaccard co-occurrence between predicted and expected labels

## How to apply

Execute the MassQL iron-bound siderophore query against the pooled LC-MS/MS dataset of paired genome–metabolome samples (the NPOmix demonstration recovered 380 putative siderophore spectra from the PoDP set). Dereplicate the MassQL hits into a nonredundant MGF of query MS/MS spectra. Build MS/MS similarity fingerprints by computing, via GNPS pairwise modified cosine scoring, the maximum similarity of each query spectrum to the spectra in each of the 1,040 paired experimental sample columns. Build the BGC training fingerprints from BiG-SCAPE outputs (similarity = 1 − raw distance; scores below the 0.7 cutoff treated as absent) and train the NPOmix k-nearest neighbor classifier with k = 3 using similarity-only features. Project each query MS/MS fingerprint into the trained KNN space, record the voted GCF labels, and report links only where predicted and expected GCF labels co-occur (Jaccard co-occurrence threshold); in the reference workflow the m/z 780 spectrum linked at 100% Jaccard to the siderophore GCF containing the Achromobacter xylosoxidans NH44784-1996 BGC.

## Related tools

- **MassQL** (Query language used to filter iron-bound siderophore spectra from the pooled LC-MS/MS profiles before dereplication and classification)
- **NPOmix** (k-nearest neighbor classifier that projects MS/MS fingerprints into a BGC-fingerprint-trained space and votes GCF labels for each query spectrum) — https://github.com/tiagolbiotech/NPOmix_python
- **BiG-SCAPE** (Computes pairwise BGC similarities (1 − raw distance) and GCF assignments used to build the BGC fingerprint training matrix (0.7 similarity cutoff)) — https://bigscape-corason.secondarymetabolites.org
- **GNPS** (Provides pairwise modified cosine scoring between query and sample MS/MS spectra to build the MS/MS fingerprint matrix; also hosts the training metabolome job) — https://gnps.ucsd.edu/ProteoSAFe/status.jsp?task=b915d1f78c364bf4be9900047760d95c
- **antiSMASH** (Produces the BGC annotations/GBK files (including siderophore-labeled BGCs such as Achromobacter xylosoxidans NH44784-1996) that constitute the training BGCs)

## Examples

```
conda activate bigscape; bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig
```

## Evaluation signals

- MassQL query recovers the expected number of putative siderophore spectra (380 in the reference run, corresponding to Dataset S1 sheet 7)
- MS/MS fingerprint matrix has the expected dimensionality of 1,040 paired-sample columns matching the training BGC matrix columns
- BiG-SCAPE-derived similarity values below the 0.7 cutoff are encoded as absent (0.0) in the training fingerprints
- The m/z 780 query spectrum is assigned to the siderophore GCF containing the Achromobacter xylosoxidans NH44784-1996 BGC with 100% Jaccard co-occurrence between predicted and expected GCF labels
- Benchmark sanity check: full NPOmix training reaches top-1 precision 81.8% and top-10 92.9% with randomness = 0 using similarity features and the Jaccard co-occurrence threshold

## Limitations

- NPOmix is correlation-based and cannot distinguish isomers or regioselectivity of homologs; new links still require proper structure elucidation (planar 2D and absolute 3D)
- Precision is not guaranteed to be reproducible for datasets dissimilar to the current PoDP-derived training set, and no RiPP-class or plant paired data exist in the training data
- Metagenome quality can hinder BGC assembly in microbiome inputs, yielding fewer, fragmented BGCs and weaker fingerprint coverage

## Evidence

- [discussion] combining NPOmix with MassQL can prioritize the discovery of putatively new siderophores: "combining NPOmix with MassQL can prioritize the discovery of putatively new siderophores"
- [discussion] precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold): "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [readme] we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample: "we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample"
- [readme] the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome: "the score of 0.0 (or below the similarity cutoff of 0.7) represents that the queried BGC is absent in the genome"
