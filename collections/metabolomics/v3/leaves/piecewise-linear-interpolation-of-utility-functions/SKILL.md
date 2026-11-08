---
name: piecewise-linear-interpolation-of-utility-functions
description: Use when aggregating GNPS, Sirius, and ISDB-LOTUS annotations of tandem
  MS/MS features into a single novelty/knownness ranking, or when expert preference
  elicitation must be encoded as piecewise-linear utility functions over score intervals.
when_to_use_negative:
- When you need a single-tool annotation confidence score (e.g. raw GNPS cosine or
  Sirius Confidence Approximate score) rather than an expert-weighted cross-tool aggregation.
- When no multi-annotation from at least two of GNPS, Sirius, and ISDB-LOTUS is available,
  since the phi agreement terms and the additive tradeoff cannot be evaluated.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2DECIDE
  role: Python library implementing the K_estimation aggregation of GNPS, ISDB-LOTUS,
    and Sirius annotations into knownness score K
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Provides FBMN library-search annotations (strict or iterative weighted analog
    search) whose match scores feed eta_g
- name: Sirius
  role: Provides structure annotations (structure_identifications.tsv, Confidence
    Approximate score) whose match scores feed eta_s
- name: ISDB-LOTUS
  role: Provides spectral-library annotations via spectral_lib_matcher/MatchMS (strict
    0.02 Da search) whose match scores feed eta_i
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:dfb95ef7f4099116a45ba87b52ecbbd9faec58d606b287211d66e08e7acf0938
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - ISDB-LOTUS
  - MS2DECIDE
  - Sirius
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# piecewise-linear-interpolation-of-utility-functions

## Summary

Reconstructs the MS2DECIDE knownness score k by encoding expert-elicited partial utility functions (eta_g, eta_s, eta_i) as piecewise-linear functions of normalized match scores with breakpoints at 0.4, 0.6, 0.8, and 1.0, plus a pairwise Tanimoto agreement term, then aggregating them via additive MAVT (Eq. 2-3, Tables 1-2).

## When to use

Use when aggregating GNPS, Sirius, and ISDB-LOTUS annotations of tandem MS/MS features into a single novelty/knownness ranking, or when expert preference elicitation must be encoded as piecewise-linear utility functions over score intervals.

## When NOT to use

- When you need a single-tool annotation confidence score (e.g. raw GNPS cosine or Sirius Confidence Approximate score) rather than an expert-weighted cross-tool aggregation.
- When no multi-annotation from at least two of GNPS, Sirius, and ISDB-LOTUS is available, since the phi agreement terms and the additive tradeoff cannot be evaluated.

## Inputs

- Quantitative feature table (.csv, e.g. MZmine >2.53 FBMN export)
- Tandem MS/MS spectra in MGF format (.mgf)
- Sirius 6 structure_identifications.tsv annotation file
- Piecewise-linear utility reference values (article Tables 1-2) for eta_g, eta_s, eta_i and phi_tau,tau0

## Outputs

- Knownness score k per ion (k_values.csv for validation against Table 3)
- Filtered, k-ranked TSV of multiannotations (.tsv from K_estimation)

## How to apply

Encode eta_g, eta_s, and eta_i as piecewise-linear functions of the normalized score tau with breakpoints at 0.4, 0.6, 0.8, and 1.0, taking the reference values from Table 1 and applying the greatest slope in the interval [0.6, 1.0], where experts judge match-score differences most important. Implement the pairwise agreement term phi_tau,tau0 of Eq. 3, activating it only when the Tanimoto similarity between a pair of annotations is >= 0.5, with reference phi values from Table 2. Enforce the tradeoff constraint eta_g(0.8) = eta_s(0.8) + eta_i(0.8) + phi_s,i(0.7, 0.8, 0.8) as a consistency check on the component functions. Compose k as the additive MAVT aggregation of eta_g, eta_s, eta_i, and the phi terms, evaluate it on the annotation pairs of the ions in Table 3, and flag mismatches beyond numerical tolerance. Handle missing annotations per the README defaults: T_gs = T_gi = 0.7 when a GNPS-Sirius Tanimoto cannot be computed, Sirius matching score = 0.5 when Sirius proposes no annotation, and treat a zero ISDB-LOTUS answer (strict 0.02 Da library search) as informative for novelty.

## Related tools

- **MS2DECIDE** (Python library implementing the K_estimation aggregation of GNPS, ISDB-LOTUS, and Sirius annotations into knownness score K) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Provides FBMN library-search annotations (strict or iterative weighted analog search) whose match scores feed eta_g)
- **Sirius** (Provides structure annotations (structure_identifications.tsv, Confidence Approximate score) whose match scores feed eta_s)
- **ISDB-LOTUS** (Provides spectral-library annotations via spectral_lib_matcher/MatchMS (strict 0.02 Da search) whose match scores feed eta_i)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Consistency check: eta_g(0.8) = eta_s(0.8) + eta_i(0.8) + phi_s,i(0.7, 0.8, 0.8) holds for the reconstructed component functions.
- Piecewise slopes match Table 1, with the greatest slope in the interval [0.6, 1.0] for all match-score utility functions.
- phi terms are zero unless the Tanimoto similarity between the annotation pair is >= 0.5; reference phi values from Table 2 are reproduced at breakpoints.
- Computed k values on the Table 3 annotation pairs match the reported k column within numerical tolerance.
- K_estimation output TSV is sorted by K and opens as a valid tab-separated file suitable for FBMN color-mapping of ranks.

## Limitations

- Default value 0.7 assigned to T_gs and T_gi when no GNPS or Sirius match exists, so the Tanimoto between them cannot be calculated — these ions carry imputed agreement values.
- Sirius missing annotations are imputed with a matching score of 0.5, which can distort eta_s for ions Sirius fails to annotate.
- Redundant features in structure_identifications.tsv raise "ValueError: DataFrame index must be unique for orient='index'."; the mappingFeatureId column must be de-duplicated.
- GNPS library score threshold of 0.001 in the workflows can cause FBMN failures; a higher threshold (e.g. 0.1) is recommended.

## Evidence

- [full_text] The match-score terms of k are piecewise linear functions broken at 0.4, 0.6, and 0.8, with the greatest slope applied in the interval [0.6, 1.0]: [evidence span withheld — anchor status: not_found]
- [readme] a default value of 0.7 was assigned to $T_{gs}$ and $T_{gi}$ in these instances: [evidence span withheld — anchor status: external_doc]
- [readme] we associated a value of 0.5 to Sirius matching score: [evidence span withheld — anchor status: external_doc]
- [abstract] leverages decision theory and expert knowledge to aggregate the outputs of three widely used annotation tools: "tandem mass spectrum to decision (MS2DECIDE) is presented which leverages decision theory and expert knowledge to aggregate the outputs of three widely used annotation tools"
- [readme] The dataframe is filtered and sorted by \( K \)-values.: [evidence span withheld — anchor status: external_doc]
