---
name: benchmark-comparison-histogram-visualization
description: Use when multiple metabolomics–genome-mining linking tools must be compared
  on identical paired genome/LC-MS/MS validation samples and their top-1 precision
  and recall plotted side by side.
when_to_use_negative:
- The dataset has no experimentally validated BGC–metabolite links — precision/recall
  cannot be scored and the benchmark histogram is meaningless
- Only one tool's output is available; a comparison histogram requires predictions
  from multiple linking tools scored identically
- The goal is reproducing NPOmix's training/prediction pipeline itself rather than
  comparing published tool performance
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NPOmix
  role: K-nearest-neighbor classifier whose reported performance (92.3% top-1 precision
    with threshold; 81.8% without) is included in the comparison
  repo: https://github.com/tiagolbiotech/NPOmix_python
- name: NPLinker
  role: Benchmarked linking tool run with the standardized Metcalf scoring scheme
    (shared rank, second-best tool)
- name: NRPminer
  role: Benchmarked linking tool run with default parameters on validation samples
- name: Nerpa
  role: Benchmarked linking tool run with default parameters on validation samples
- name: GARLIC
  role: Benchmarked linking tool run with default parameters on validation samples
- name: antiSMASH
  role: Provides the BGC annotations (.gbk) that each benchmarked tool links to MS/MS
    spectra
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/pnasnexus/pgac257
    title: 'NPOmix: A machine learning classifier to connect mass spectrometry fragmentation
      data to biosynthetic gene clusters'
  source_content_hash: sha256:7b063bbf4df9d1c62e9a95674fb9362fce0a006e3608be73a04f611905f737d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tiagolbiotech/NPOmix_python
  tools:
  - GARLIC
  - NPLinker
  - NPOmix
  - NRPminer
  - Nerpa
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7727272727272727
n_claims: 22
---

# benchmark-comparison-histogram-visualization

## Summary

Benchmark BGC–MS/MS linking tools (NPOmix, NPLinker, NRPminer, Nerpa, GARLIC) on the same 22 experimentally validated links from the Paired Omics Data Platform and render a grouped precision/recall histogram reproducing Fig. 4 of the NPOmix paper.

## When to use

Use when multiple metabolomics–genome-mining linking tools must be compared on identical paired genome/LC-MS/MS validation samples and their top-1 precision and recall plotted side by side.

## When NOT to use

- The dataset has no experimentally validated BGC–metabolite links — precision/recall cannot be scored and the benchmark histogram is meaningless
- Only one tool's output is available; a comparison histogram requires predictions from multiple linking tools scored identically
- The goal is reproducing NPOmix's training/prediction pipeline itself rather than comparing published tool performance

## Inputs

- Paired genome + LC-MS/MS validation samples (9 samples) from the Paired Omics Data Platform
- GNPS job output for task=b915d1f78c364bf4be9900047760d95c
- antiSMASH BGC annotations (.gbk files, e.g. antismash_only_gbk.zip from Zenodo record 6637083)
- Set of 22 experimentally validated BGC–metabolite links
- Predicted BGC–metabolite links from each benchmarked tool (NPOmix, NPLinker with Metcalf scoring, NRPminer, Nerpa, GARLIC)

## Outputs

- Per-tool top-1 precision and top-1 recall values on the 22 validated links
- Note of unavailable/discontinued tools (benchmark table analogue of Table S3)
- Grouped histogram figure4_tool_comparison_histogram.png matching Fig. 4 layout

## How to apply

Retrieve the paired validation samples (genome + LC-MS/MS) for the 9 samples containing the 22 experimentally validated BGC–metabolite links, using the Paired Omics Data Platform records and the GNPS job task=b915d1f78c364bf4be9900047760d95c. Run each linking tool — NRPminer, NPLinker (with the standardized Metcalf scoring scheme), Nerpa, GARLIC — with default parameters on each validation sample's antiSMASH BGCs and MS/MS spectra, recording tools that could not be executed (as in Table S3). Parse each tool's predicted links and score them against the 22 validated links to compute top-1 precision and top-1 recall per tool. Combine the computed metrics with NPOmix's best reported run — top-1 precision 92.3% with 100% recall, obtained using similarity, substructure, and biosynthetic class features with a threshold — and plot a grouped histogram (figure4_tool_comparison_histogram.png) matching the Fig. 4 layout.

## Related tools

- **NPOmix** (K-nearest-neighbor classifier whose reported performance (92.3% top-1 precision with threshold; 81.8% without) is included in the comparison) — https://github.com/tiagolbiotech/NPOmix_python
- **NPLinker** (Benchmarked linking tool run with the standardized Metcalf scoring scheme (shared rank, second-best tool))
- **NRPminer** (Benchmarked linking tool run with default parameters on validation samples)
- **Nerpa** (Benchmarked linking tool run with default parameters on validation samples)
- **GARLIC** (Benchmarked linking tool run with default parameters on validation samples)
- **antiSMASH** (Provides the BGC annotations (.gbk) that each benchmarked tool links to MS/MS spectra)

## Examples

```
python plot_grouped_histogram.py --validated_links 22 --metrics npomix=92.3,100 nplinker metcalf_top1 --output figure4_tool_comparison_histogram.png
```

## Evaluation signals

- figure4_tool_comparison_histogram.png exists as the grouped histogram output
- Computed top-1 precision/recall values for each tool are consistent with the values reported in the article's Fig. 4 benchmark comparison
- NPOmix appears as the highest-precision tool (92.3% with similarity, substructure, and biosynthetic class features and threshold) together with 100% recall, above NPLinker with shared rank
- Tools that could not be executed are noted rather than silently dropped, mirroring Table S3
- All per-tool metrics are computed against exactly the same 22 validated links from the 9 paired validation samples

## Limitations

- Some linking tools are unavailable or discontinued and cannot be benchmarked; they must be flagged in the comparison table
- The article's reported precision (e.g. top-1 = 81.8% and top-10 = 92.9% under a different feature set) may not be reproducible for datasets dissimilar to the PoDP training set
- Correlation-based linking approaches such as NPOmix do not have the resolution to distinguish between isomers and regioselectivity of homologs, which affects link-level ground truth

## Evidence

- [other] In the tool comparison, NPOmix achieved the highest precision (92.3%, using similarity, substructure, and biosynthetic class features with threshold) together with 100% recall, outperforming the second-best tool, NPLinker with shared rank.: [evidence span withheld — anchor status: not_found]
- [discussion] Article discusses NPOmix precision under the Jaccard co-occurrence threshold.: "precision scores of top-1 = 81.8% and top-10 = 92.9% (both with randomness equal to 0, using the similarity and biosynthetic class features and using the Jaccard co-occurrence threshold)"
- [intro] Adding biosynthetic class and substructure features improves the performance of our Natural Products Mixed Omics tool: [evidence span withheld — anchor status: not_found]
- [readme] README explains BGC fingerprint similarity scores ranging from 0.0 to 1.0 with a 0.7 cutoff used when benchmarking BGC inputs.: "Similarity scores range from 0.0 to 1.0; for instance, identical BGCs have a perfect similarity score of 1.0, a score of 0.8 would represent that a homologous BGC is present in the genome, and the"
