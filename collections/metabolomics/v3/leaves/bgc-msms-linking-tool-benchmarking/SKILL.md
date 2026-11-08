---
name: bgc-msms-linking-tool-benchmarking
description: Use when comparing biosynthetic gene cluster-to-mass-spectra linking
  performance of multiple published tools against a gold-standard set of validated
  BGC–metabolite pairs from paired omics data.
when_to_use_negative:
- The dataset lacks experimentally validated BGC–metabolite links, so there is no
  gold standard against which to compute precision/recall
- You only need to run one linking tool on new samples rather than compare multiple
  tools head-to-head — use the NPOmix prediction workflow instead
- The input genomes have no BGCs similar to the PoDP training set (e.g. fungal species),
  where tools like NPOmix produce no output at all
edam_topics:
- http://edamontology.org/topic_3364
- http://edamontology.org/topic_0622
tools:
- name: NPOmix
  role: Machine learning (k-NN) linking tool whose reported performance (precision
    92.3%, recall 100%) anchors the benchmark comparison
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: NPLinker
  role: Comparison tool benchmarked with the standardized Metcalf scoring scheme;
    second-best performer with shared rank
- name: NRPminer
  role: Comparison tool benchmarked with default parameters on the validation samples
- name: Nerpa
  role: Comparison tool benchmarked with default parameters on the validation samples
- name: GARLIC
  role: Comparison tool benchmarked with default parameters on the validation samples
- name: antiSMASH
  role: Produces the BGC GenBank annotations supplied to each linking tool for the
    validation genomes
  repo: https://antismash.secondarymetabolites.org/#!/start
- name: BiG-SCAPE
  role: Computes pairwise BGC similarity scores and GCF assignments used in fingerprint
    construction for the classifier
  repo: https://bigscape-corason.secondarymetabolites.org
- name: GNPS
  role: Source of the paired LC-MS/MS metabolome job (task=b915d1f78c364bf4be9900047760d95c)
    and modified cosine scoring
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:b41ad65d7961bf327f031d672d93122b140869d406420d79b1c3767f81048169
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - BiG-SCAPE
  - GARLIC
  - GNPS
  - NPLinker
  - NPOmix
  - NRPminer
  - Nerpa
  - antiSMASH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3364
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 20
---

# bgc-msms-linking-tool-benchmarking

## Summary

Benchmarks multi-omics BGC–MS/MS linking tools (NPOmix, NPLinker, NRPminer, Nerpa, GARLIC) on paired genome–LC-MS/MS samples with experimentally validated links, computing top-1 precision and recall per tool and visualizing them as a grouped histogram.

## When to use

Use when comparing biosynthetic gene cluster-to-mass-spectra linking performance of multiple published tools against a gold-standard set of validated BGC–metabolite pairs from paired omics data.

## When NOT to use

- The dataset lacks experimentally validated BGC–metabolite links, so there is no gold standard against which to compute precision/recall
- You only need to run one linking tool on new samples rather than compare multiple tools head-to-head — use the NPOmix prediction workflow instead
- The input genomes have no BGCs similar to the PoDP training set (e.g. fungal species), where tools like NPOmix produce no output at all

## Inputs

- Paired genome/metagenome + LC-MS/MS validation samples (9 samples, 22 validated BGC–metabolite links) from the Paired Omics Data Platform
- GNPS job output for task=b915d1f78c364bf4be9900047760d95c
- antiSMASH BGC GenBank files (e.g. antismash_only_gbk.zip from Zenodo record 6637083)
- BiG-SCAPE similarity network outputs for antiSMASH BGCs (concatenated across classes, with --mibig)
- Predicted BGC–metabolite links from NRPminer, NPLinker (Metcalf scoring), Nerpa, and GARLIC

## Outputs

- Top-1 precision and top-1 recall per linking tool scored against the 22 validated links
- Benchmark comparison table (noting tools that could not be executed, as in Table S3)
- Grouped histogram comparing tool performance (figure4_tool_comparison_histogram.png)

## How to apply

Retrieve paired validation samples (genome + LC-MS/MS) covering the validated links from the Paired Omics Data Platform and the GNPS job task=b915d1f78c364bf4be9900047760d95c. Run each linking tool (NRPminer, NPLinker with the standardized Metcalf scoring scheme, Nerpa, GARLIC) with default parameters on each sample's antiSMASH BGCs and MS/MS spectra, noting any tools that cannot be executed as in Table S3. Parse each tool's predicted links, score them against the 22 validated links to compute top-1 precision and top-1 recall, then combine with NPOmix's reported performance and plot a grouped histogram. NPOmix itself uses BiG-SCAPE similarity scores (1 minus raw distance, with a similarity cutoff of 0.7) to build BGC fingerprints and GNPS modified cosine scores for MS/MS fingerprints, fed to a k-nearest neighbor classifier with GCF labels.

## Related tools

- **NPOmix** (Machine learning (k-NN) linking tool whose reported performance (precision 92.3%, recall 100%) anchors the benchmark comparison) — https://github.com/tiagolbiotech/NPOmix_python
- **NPLinker** (Comparison tool benchmarked with the standardized Metcalf scoring scheme; second-best performer with shared rank)
- **NRPminer** (Comparison tool benchmarked with default parameters on the validation samples)
- **Nerpa** (Comparison tool benchmarked with default parameters on the validation samples)
- **GARLIC** (Comparison tool benchmarked with default parameters on the validation samples)
- **antiSMASH** (Produces the BGC GenBank annotations supplied to each linking tool for the validation genomes) — https://antismash.secondarymetabolites.org/#!/start
- **BiG-SCAPE** (Computes pairwise BGC similarity scores and GCF assignments used in fingerprint construction for the classifier) — https://bigscape-corason.secondarymetabolites.org
- **GNPS** (Source of the paired LC-MS/MS metabolome job (task=b915d1f78c364bf4be9900047760d95c) and modified cosine scoring)

## Examples

```
bigscape.py --pfam_dir /path/to/pfam_files/ -i /path/to/antismash/ -o /path/to/bigscape_outputs_220603_1791samples/ -c 8 --include_singletons --mibig
```

## Evaluation signals

- Computed top-1 precision/recall for each tool match the values shown in the article's Fig. 4 grouped histogram
- NPOmix's combined-features condition reproduces 92.3% precision with 100% recall, ranked above NPLinker (second best, shared rank)
- figure4_tool_comparison_histogram.png exists and matches the layout of Fig. 4
- Tools that are unavailable/discontinued are explicitly listed in the benchmark table rather than silently dropped (per Table S3 convention)
- Precision is scored only against the 22 experimentally validated links, not against unvalidated predictions

## Limitations

- Precision scores are dataset-dependent and may not be reproducible when the evaluation set differs substantially from the PoDP-derived training set
- Correlation-based linking cannot distinguish isomers or regioselectivity among homologs, capping achievable precision on such targets
- NPOmix requires similar BGCs in the training data: a fungal species with no BGC similar to the training set yields no output; the PoDP validation data also lack RiPP-class metabolites and plant paired data

## Evidence

- [discussion] precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold): "precision scores of top-1 = 81.8% and top-10 = 92.9%"
- [abstract] Using paired datasets that include validated genes-mass spectral links from the Paired Omics Data Platform: "Using paired datasets that include validated genes-mass spectral links from the Paired Omics Data Platform"
- [discussion] The reported precision is dataset-dependent and cannot be assumed transferable.: "the precision score may not be reproducible for every dataset, especially if the dataset is not too similar to the current training set"
- [readme] BiG-SCAPE similarity scores and 0.7 cutoff underpin BGC fingerprint construction used in the benchmarked classifier.: "By applying BiG-SCAPE, each biosynthetic gene cluster (BGC) in the genomes will go through a pairwise similarity comparison"
- [readme] Modified cosine scoring between query and per-sample spectra defines the MS/MS fingerprint side of the benchmark.: "we used GNPS to calculate the pairwise modified cosine score and then identify the maximum similarity of the query MS/MS spectrum to one of the many MS/MS spectra in each experimental sample"
