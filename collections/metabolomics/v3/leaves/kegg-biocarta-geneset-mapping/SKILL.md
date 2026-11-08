---
name: kegg-biocarta-geneset-mapping
description: Use when a microarray-style expression matrix must be partitioned into
  predefined pathway gene sets (KEGG/Biocarta) before computing pathway activity scores
  or differential pathway rankings.
when_to_use_negative:
- The question concerns differential expression of individual genes rather than predefined
  pathway-level activity (the article positions the pathway-based results as complementary
  to gene-focused analyses).
- The dataset cannot be mapped onto KEGG/Biocarta gene identifiers (e.g. non-standard
  gene labels with no cross-reference), since pathway-restricted matrices cannot be
  formed.
- A gene-set method other than the KEGG/Biocarta collections is required, e.g. custom
  or GO-term gene sets not present among the ~400 pathways used here.
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PLAGE
  role: web program that performs the SVD-based pathway activity analysis over the
    mapped KEGG/Biocarta gene sets, including permutation-based significance testing
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:5a52a9dacb1ed3eb9fcda721121eb3114521ca5552dd2f253edc16c8a55bcbf2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# kegg-biocarta-geneset-mapping

## Summary

Map a gene expression dataset onto the ~400 predefined KEGG and Biocarta pathways/complexes used by the PLAGE method, so that each pathway can be assigned an SVD-derived activity level per sample and tested for differential activity.

## When to use

Use when a microarray-style expression matrix must be partitioned into predefined pathway gene sets (KEGG/Biocarta) before computing pathway activity scores or differential pathway rankings.

## When NOT to use

- The question concerns differential expression of individual genes rather than predefined pathway-level activity (the article positions the pathway-based results as complementary to gene-focused analyses).
- The dataset cannot be mapped onto KEGG/Biocarta gene identifiers (e.g. non-standard gene labels with no cross-reference), since pathway-restricted matrices cannot be formed.
- A gene-set method other than the KEGG/Biocarta collections is required, e.g. custom or GO-term gene sets not present among the ~400 pathways used here.

## Inputs

- gene expression matrix (rows = genes, columns = samples), e.g. the AGED airway epithelia data from 75 subjects (34 current, 18 former, 23 never smokers)
- predefined KEGG pathway gene sets
- predefined Biocarta pathway/complex gene sets
- sample group labels for pairwise comparisons

## Outputs

- pathway-restricted expression matrix Y for each of the ~400 KEGG/Biocarta pathways
- per-sample pathway activity levels (first-row coefficients cj of C from the SVD)
- ranked table of top differentially expressed pathways with p-values (e.g. table2_top_pathways.tsv)

## How to apply

Retrieve the pathway gene-set definitions from the KEGG and Biocarta websites, matching the versions used for the analysis (the PLAGE article notes its versions were posted at the PLAGE website). For each of the ~400 pathways and complexes, restrict the standardized expression matrix (genes standardized to zero mean and unit variance over samples) to the pathway's genes, forming the pathway-restricted matrix Y whose rows are genes and columns are samples. Perform the singular value decomposition Y = WDC and record the first-row coefficients cj of C as the pathway's activity levels per sample; pathways with too few genes measured in the dataset cannot form a meaningful metagene and should be excluded. These activity levels then feed pairwise two-group t statistics (e.g., current vs never smokers) with significance from 10,000 label permutations, recording the maximum t-statistic per permutation.

## Related tools

- **PLAGE** (web program that performs the SVD-based pathway activity analysis over the mapped KEGG/Biocarta gene sets, including permutation-based significance testing) — http://dulci.biostat.edu/pathways

## Examples

```
python -c "import numpy as np; Y = X.loc[genes_in_pathway].values; W,D,C = np.linalg.svd(Y, full_matrices=False); activity = C[0]"
```

## Evaluation signals

- The mapped gene sets cover approximately 400 pathways and complexes drawn from the Biocarta and KEGG websites, matching the article's stated collection size.
- For each pathway, the pathway-restricted matrix Y decomposes as Y = WDC, and the recorded activity vector has one value per sample (dimension equals number of samples).
- Applying the t-statistic with 10,000 random label permutations should rank 'gamma-Hexachlorocyclohexane degradation' (KEGG) as the most significant pathway in both current-vs-never and former-vs-never smoker comparisons for the smoking dataset.
- Permutation p-values are computed as the fraction of maximum t-statistics (across pathways per permutation) exceeding each pathway's observed t-statistic, so p-values must be multiples of at least 1/10,000.
- Output table reproduces the Table 2 rankings with pathway names qualified by source (KEGG or Biocarta).

## Limitations

- KEGG and Biocarta pathways continually expand and evolve; the exact pathway versions used are not version-numbered in the article and must be obtained from the PLAGE website to reproduce results.
- The t-statistic formula is referenced via an equation placeholder and is not fully rendered in the extracted text, so exact test details must be sourced from the full article.
- Significance assessment requires a large number of permutations (10,000 in the article), which dominates runtime.

## Evidence

- [discussion] about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [discussion] pathway versions: "The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11]."
- [intro] SVD activity levels: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [discussion] permutation scheme: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
