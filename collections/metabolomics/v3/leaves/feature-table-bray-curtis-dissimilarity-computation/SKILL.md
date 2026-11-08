---
name: feature-table-bray-curtis-dissimilarity-computation
description: Use when an aligned GNPS FBMN-style feature abundance table is available
  and you need a composition-only baseline distance to quantify batch effects (e.g.,
  RT-shift pseudo-F) against MS2/fragmentation-aware methods.
when_to_use_negative:
- When samples were acquired with different LC methods or have a strong RT shift /
  poor feature overlap — Bray-Curtis on aligned features then inflates the batch effect
  (pseudo-F 113.55) instead of mitigating it
- When no prior feature alignment is possible or desired — MEMO's MS2 fingerprinting
  was designed to compare samples without a prior feature alignment step and in significantly
  shorter computational time
- When chemical structural relatedness of features should be accounted for — CSCS
  integrates cosine scores from mass spectral molecular networks, which Bray-Curtis
  ignores
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: scikit-bio
  role: Computes the Bray-Curtis dissimilarity matrix from the feature abundance table
    and runs PERMANOVA (999 permutations) on it for the experiment and composition
    factors
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3389/fbinf.2022.842964
    title: 'MEMO: Mass Spectrometry-Based Sample Vectorization to Explore Chemodiverse
      Datasets'
  source_content_hash: sha256:f96d7beb14c839c07c7fc0bd973b910340b1eecbdd443137e8694478b9246c4b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - matchms
  - numpy
  - scikit-bio
  - spec2vec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# feature-table-bray-curtis-dissimilarity-computation

## Summary

Computes pairwise Bray-Curtis dissimilarity directly from a mass spectral feature abundance table using scikit-bio, serving as the feature-table baseline in the MEMO benchmark against MS2-based sample clustering (MEMO, Qemistree UniFrac, CSCS) via PERMANOVA pseudo-F statistics.

## When to use

Use when an aligned GNPS FBMN-style feature abundance table is available and you need a composition-only baseline distance to quantify batch effects (e.g., RT-shift pseudo-F) against MS2/fragmentation-aware methods.

## When NOT to use

- When samples were acquired with different LC methods or have a strong RT shift / poor feature overlap — Bray-Curtis on aligned features then inflates the batch effect (pseudo-F 113.55) instead of mitigating it
- When no prior feature alignment is possible or desired — MEMO's MS2 fingerprinting was designed to compare samples without a prior feature alignment step and in significantly shorter computational time
- When chemical structural relatedness of features should be accounted for — CSCS integrates cosine scores from mass spectral molecular networks, which Bray-Curtis ignores

## Inputs

- Feature abundance table (e.g., GNPS FBMN bucket table) with samples as columns/rows and mass spectral features as variables
- Sample metadata mapping samples to the 'experiment' factor (C18 vs C18 RT-shift) and 'composition' factor
- scikit-bio DistanceMatrix-compatible metadata objects

## Outputs

- Symmetric 27x27 Bray-Curtis dissimilarity matrix
- PERMANOVA pseudo-F statistics for 'experiment' and 'composition' factors (999 permutations)
- Rows for the comparison table written to permanova_results.csv

## How to apply

Load the feature abundance table (e.g., the GNPS FBMN bucket table from the Qemistree evaluation dataset, 27 samples acquired with C18 and C18 RT-shift LC methods on a Q-Exactive instrument) and compute the Bray-Curtis dissimilarity matrix directly on feature abundances with scikit-bio. Then run PERMANOVA with 999 permutations (scikit-bio permanova) on this matrix for both the 'experiment' factor and the 'composition' factor to obtain pseudo-F statistics. In the MEMO benchmark this feature-table baseline yielded an experiment-factor pseudo-F of 113.55 versus <=81.64 for MS2-based methods, and a composition pseudo-F of only 4.72 versus >=11.10 for fragmentation-spectrum-based distances, demonstrating that Bray-Curtis on aligned features inflates the retention-time batch effect while under-discriminating samples. Write the resulting pseudo-F values alongside the six clustering methods into a comparison table (permanova_results.csv) to quantify how poorly the feature table mitigates the RT-induced batch effect.

## Related tools

- **scikit-bio** (Computes the Bray-Curtis dissimilarity matrix from the feature abundance table and runs PERMANOVA (999 permutations) on it for the experiment and composition factors)

## Examples

```
from skbio.stats.distance import permanova; permanova(dm_braycurtis, metadata, column='experiment', permutations=999)
```

## Evaluation signals

- On the Qemistree evaluation dataset (27 samples, C18 vs C18 RT-shift, Q-Exactive), the experiment-factor pseudo-F should approximate the reported 113.55, clearly higher than the <=81.64 obtained by MS2-based methods
- The composition-factor pseudo-F should approximate 4.72, lower than the >=11.10 obtained by fragmentation-spectrum-based distances
- The Bray-Curtis distance matrix must be symmetric with a zero diagonal and values in [0, 1]
- PERMANOVA must use 999 permutations, matching the benchmark protocol, and results must be reproducible in the comparison written to permanova_results.csv
- The computed baseline should rank last among the six tested methods for RT-batch-effect mitigation, consistent with Table 1

## Limitations

- Bray-Curtis on an aligned feature table is retention-time agnostic only after alignment; it inherits alignment artifacts and inflates the RT-induced batch effect in the benchmark
- It ignores structural relatedness among mass spectral features (cosine scores), under-discriminating samples by composition (pseudo-F 4.72)
- In PCoA space, Bray-Curtis can fail to separate samples sharing structurally related compounds, as shown in the q2-cscs unit-test example

## Evidence

- [methods] conda install -c conda-forge scikit-bio: "conda install -c conda-forge scikit-bio"
- [readme] Bray-Curtis fails to differentiate structurally related samples in PCoA: [evidence span withheld — anchor status: external_doc]
- [abstract] MEMO's advantage over feature-table approaches: no alignment needed: "such performance was achieved without the requirement of a prior feature alignment step and in a significantly shorter computational time"
- [intro] MEMO targets chemodiverse samples with poor feature overlap or strong RT shift: "MEMO suits particularly well to compare chemodiverse samples, ie with a
poor features overlap, or to compare samples with a strong RT shift"
