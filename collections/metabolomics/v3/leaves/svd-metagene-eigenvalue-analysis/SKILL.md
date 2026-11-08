---
name: svd-metagene-eigenvalue-analysis
description: Use when you have gene expression matrices over groups (e.g. 17 DM2/17
  NGT/9 IGT diabetes samples or 75 airway epithelium smoker samples) and need to check
  whether gene-set enrichment rankings reflect coordinate expression strength rather
  than consistent group differences.
when_to_use_negative:
- Your goal is a straightforward differential pathway-activity test (permute sample
  labels, record the max t-statistic across pathways) rather than diagnosing whether
  enrichment ranks are driven by coordinate expression strength.
- Your pathway gene-sets are very small or near-uncorrelated by construction — the
  first-eigenvalue fraction will sit near the ~0.05 size-matched Gaussian noise baseline
  and carry little signal.
- You lack the raw gene expression matrix and only have precomputed enrichment scores,
  since the SVD must be computed on the restricted, standardized gene-by-sample matrix
  Y.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: PLAGE
  role: Web program implementing the SVD-based pathway level analysis of gene expression;
    used to derive pathway activity levels and the pathway statistics compared against
    GSEA
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:a167449a1bb80ff4677bcac01ca2df8a66ad08f78c7ac1512944dd96e25e2a61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# svd-metagene-eigenvalue-analysis

## Summary

Quantifies how strongly each ~400-KEGG/Biocarta pathway gene-set is coordinately expressed by computing the first-eigenvalue fraction of the SVD of its standardized gene-by-sample expression matrix, and joins this metric to GSEA enrichment scores to test whether GSEA's top-ranked sets are merely the most coordinated (e.g. the ribosome set) rather than the most differentially active.

## When to use

Use when you have gene expression matrices over groups (e.g. 17 DM2/17 NGT/9 IGT diabetes samples or 75 airway epithelium smoker samples) and need to check whether gene-set enrichment rankings reflect coordinate expression strength rather than consistent group differences.

## When NOT to use

- Your goal is a straightforward differential pathway-activity test (permute sample labels, record the max t-statistic across pathways) rather than diagnosing whether enrichment ranks are driven by coordinate expression strength.
- Your pathway gene-sets are very small or near-uncorrelated by construction — the first-eigenvalue fraction will sit near the ~0.05 size-matched Gaussian noise baseline and carry little signal.
- You lack the raw gene expression matrix and only have precomputed enrichment scores, since the SVD must be computed on the restricted, standardized gene-by-sample matrix Y.

## Inputs

- Type 2 diabetes expression profiles (43 samples: 17 DM2, 17 NGT, 9 IGT) from the Whitehead Institute Center for Genome Research site
- Airway epithelium expression profiles (75 subjects: 34 current, 18 former, 23 never smokers) from the Airway Gene Expression Database (AGED)
- Pathway gene-set definitions: ~400 pathways and complexes from the KEGG and Biocarta websites (versions used available at the PLAGE website)

## Outputs

- Per-pathway first-eigenvalue fraction lambda^2/(ng*ns) of total variation for every pathway in both datasets
- eigenvalue_fraction_vs_gsea_rank.csv — table joining each pathway's first-eigenvalue fraction to its GSEA ES and rank in each of the four two-group comparisons

## How to apply

Standardize every gene expression level to zero mean and unit variance over samples, then for each pathway form the restricted gene-by-sample matrix Y (rows = pathway genes ng, columns = samples ns) and compute its SVD Y = WDC. Record the first eigenvalue and compute the first-eigenvalue fraction lambda^2/(ng*ns) as the fraction of total variation explained by the pathway's first metagene. Benchmark each pathway's fraction against expectations: ~0.53 for the ribosome set in the smoking data, ~0.23 for Oxidative Phosphorylation, and ~0.05 for uncorrelated Gaussian noise of ribosome size averaged over 1000 simulated sets. In parallel, run GSEA on the same pathway collection over the four two-group comparisons, ranking genes by signal-to-noise ratio with 1000 random permutations, then assemble a table joining each pathway's first-eigenvalue fraction to its GSEA ES and rank in each comparison (written to eigenvalue_fraction_vs_gsea_rank.csv). The rationale, following PLAGE's SVD-based pathway-activity framing, is that sets built from coordinated gene groups behave as robust features, so a high first-eigenvalue fraction indicates a dominant metagene that can dominate enrichment scores without distinguishing group differences.

## Related tools

- **PLAGE** (Web program implementing the SVD-based pathway level analysis of gene expression; used to derive pathway activity levels and the pathway statistics compared against GSEA) — http://dulci.biostat.edu/pathways

## Examples

```
import numpy as np; Y = expr.loc[genes].values; Y = (Y - Y.mean(0)) / Y.std(0); U, S, Vh = np.linalg.svd(Y); frac = S[0]**2 / (Y.shape[0] * Y.shape[1])
```

## Evaluation signals

- Ribosome gene-set's first metagene accounts for ~0.53 (over fifty percent) of total variation in the smoking dataset.
- Oxidative Phosphorylation's first-eigenvalue fraction is ~0.23, while uncorrelated Gaussian noise of ribosome size averaged over 1000 simulated sets yields ~0.05.
- Ribosome gene-set has the highest enrichment score in three of the four GSEA comparisons (e.g. ES=304, p=0.047 former-vs-never; ES=278, p=0.073 DM2-vs-NGT), with the expected top-ES sets reproduced for the other comparisons (Lectin Induced Complement Pathway ES=128 p=0.52 current-vs-former; OxPhos ES=261 p=0.092 DM2-vs-NGT).
- eigenvalue_fraction_vs_gsea_rank.csv contains one row per pathway with joined first-eigenvalue fraction, GSEA ES, and rank for each of the four comparisons.
- All genes standardized to zero mean and unit variance over samples before forming each pathway-restricted matrix Y.

## Limitations

- The KEGG and Biocarta pathway collections continue to expand and evolve; no specific version numbers or accessions are pinned, though the versions used are available at the PLAGE website.
- A high first-eigenvalue fraction indicates strong coordinate expression, which GSEA may reward with high enrichment scores without the set showing consistent group differences — this is a diagnostic caveat, not evidence of differential activity.
- A t-statistic formula is referenced in the article but not rendered in the available text, so exact per-pathway test details cannot be reproduced from this section alone.

## Evidence

- [intro] Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD).: "Our method instead begins by translating gene expression levels into pathway 'activity' levels, which are derived from singular value decompositions (SVD)."
- [discussion] about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites: "about 400 pathways and complexes characterized on the Biocarta and KEGG (Kyoto Encyclopedia of Genes and Genomes) websites"
- [discussion] The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11].: "The KEGG and Biocarta pathways continue to expand and evolve. The versions used for this article are available at the PLAGE website [11]."
- [discussion] For one, features defined by groups of genes will tend to be more robust in the face of variation at the level of individual genes.: "For one, features defined by groups of genes will tend to be more robust in the face of variation at the level of individual genes."
- [abstract] A comparison with results from the related approach, 'gene-set enrichment analysis', is also provided.: "A comparison with results from the related approach, 'gene-set enrichment analysis', is also provided."
