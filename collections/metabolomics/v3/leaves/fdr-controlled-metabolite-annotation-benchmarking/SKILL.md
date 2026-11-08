---
name: fdr-controlled-metabolite-annotation-benchmarking
description: Use when comparing FDR-controlled metabolite annotation methods (e.g.,
  ML vs rule-based scorers) across imaging MS datasets, or when evaluating target/decoy
  ranking quality via average precision.
when_to_use_negative:
- The goal is to evaluate database choice (e.g., CoreMetabolome vs general databases)
  rather than ranking-model comparison — MAP against databases is a related but distinct
  analysis
- Input datasets are QTOF-based — such datasets are underrepresented in the METASPACE-ML
  model due to lack of publicly available data, making evaluation unreliable
- The task is single-dataset annotation rather than method benchmarking across a testing
  corpus — use the METASPACE engine directly instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: METASPACE-ML
  role: Machine-learning (GBDT) scorer producing FDR-controlled target/decoy ion rankings
    to be benchmarked against MSM
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Engine used to obtain target and decoy annotation candidates scored by both
    METASPACE-ML and the rule-based MSM scorer
  repo: https://github.com/metaspace2020/metaspace
- name: Lithops
  role: Serverless cloud computing framework providing efficient implementation for
    scoring large numbers of testing datasets
  repo: https://github.com/metaspace2020/metaspace
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:d9beda463368e857cdbb7925f43a053623b522cfa5d04101077218c1e2260b18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - Lithops
  - METASPACE
  - METASPACE-ML
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# fdr-controlled-metabolite-annotation-benchmarking

## Summary

Benchmarks METASPACE-ML, a machine-learning metabolite annotation method for imaging mass spectrometry, against the rule-based MSM scorer by ranking target-versus-decoy ion candidates and computing mean average precision (MAP) per sample type.

## When to use

Use when comparing FDR-controlled metabolite annotation methods (e.g., ML vs rule-based scorers) across imaging MS datasets, or when evaluating target/decoy ranking quality via average precision.

## When NOT to use

- The goal is to evaluate database choice (e.g., CoreMetabolome vs general databases) rather than ranking-model comparison — MAP against databases is a related but distinct analysis
- Input datasets are QTOF-based — such datasets are underrepresented in the METASPACE-ML model due to lack of publicly available data, making evaluation unreliable
- The task is single-dataset annotation rather than method benchmarking across a testing corpus — use the METASPACE engine directly instead

## Inputs

- Supplementary Data 5 list of testing datasets (720 animal, 210 plant imaging MS datasets)
- Target and decoy annotation candidates per dataset with scores from METASPACE-ML and rule-based MSM
- Sample type labels (animal, plant) for grouping and cross-validated MAP aggregation

## Outputs

- map_results.csv with per-dataset AP values for both methods
- Median MAP values per sample type per method (0.36 vs 0.27 animal; 0.32 vs 0.17 plant)
- Counts of datasets with statistically significant AP differences (674/720 animal; 207/210 plant)

## How to apply

Load the evaluation set of testing datasets (720 animal, 210 plant from Supplementary Data 5) and, for each dataset, obtain target and decoy annotation candidates scored by both METASPACE-ML and the rule-based MSM scorer using the METASPACE engine. Rank target and decoy ions per dataset by each method's score, compute average precision (AP) per dataset per method, and aggregate AP values into mean average precision (MAP) within each sample type. Compare median MAP values between methods (expected: 0.36 vs 0.27 animal; 0.32 vs 0.17 plant) and count datasets where the AP difference is statistically significant (expected 674/720 animal, 207/210 plant). Rationale: METASPACE-ML's GBDT ranking model, which is robust to noisy outliers affecting target/decoy separation, should outperform the MSM scorer that assigns equal weights to features. Write per-dataset AP values plus aggregated medians and significance counts to map_results.csv.

## Related tools

- **METASPACE-ML** (Machine-learning (GBDT) scorer producing FDR-controlled target/decoy ion rankings to be benchmarked against MSM) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Engine used to obtain target and decoy annotation candidates scored by both METASPACE-ML and the rule-based MSM scorer) — https://github.com/metaspace2020/metaspace
- **Lithops** (Serverless cloud computing framework providing efficient implementation for scoring large numbers of testing datasets) — https://github.com/metaspace2020/metaspace

## Examples

```
python benchmark_map.py --datasets supplementary_data5.csv --methods metaspace_ml,msm --output map_results.csv
```

## Evaluation signals

- Computed median MAP for METASPACE-ML vs MSM reproduces 0.36 vs 0.27 on animal and 0.32 vs 0.17 on plant cross-validated testing datasets
- Count of datasets with significantly higher median AP matches 674/720 animal and 207/210 plant testing datasets
- map_results.csv contains one AP value per dataset per method covering the full 720 + 210 dataset corpus
- Per-dataset AP values are bounded in [0, 1] and computed on rankings of both target and decoy ions

## Limitations

- MSM's sensitivity to the rho_spatial feature score means some annotations are captured by MSM but not METASPACE-ML, so MAP comparison may understate complementary annotations
- Decoy ions show heterogeneity in similarity to target ions; the most reliable decoy generation method needs further investigation, affecting FDR estimation
- Training/evaluation data are biased towards public METASPACE datasets (from 47 labs); less common data types such as single-cell metabolomics or industrial lab data are underrepresented
- rho_chaos6 had only minor impact on model performance — target-decoy separation is mainly driven by rho_spectral

## Evidence

- [results] METASPACE-ML achieved higher median MAP scores than the rule-based approach (0.36 vs 0.27 on animal and 0.32 vs 0.17 on plant cross-validated datasets): [evidence span withheld — anchor status: not_found]
- [discussion] GBDT robustness to noisy outliers: "GBDT is known to be robust to noisy outliers, which can significantly impact the separation of target and decoy ions"
- [intro] MSM equal-weight limitation: "a limitation of METASPACE is its rule-based scoring system, namely Metabolite Signal Matching (MSM), which assigns equal weights to the features and lacks adaptability to data variations"
- [readme] README: METASPACE engine description: "The METASPACE platform hosts an engine for
 metabolite annotation of imaging mass spectrometry data as well as a
 spatial metabolite knowledgebase"
- [abstract] ML outperformance claim: "METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity and biologically-relevant metabolites."
