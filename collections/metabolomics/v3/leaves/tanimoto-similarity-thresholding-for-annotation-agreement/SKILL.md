---
name: tanimoto-similarity-thresholding-for-annotation-agreement
description: Use when aggregating multi-annotations of MS/MS spectra and a pairwise
  structural-agreement term is needed in the knownness score k, including cases where
  a tool returned no annotation.
when_to_use_negative:
- When only a single annotation tool is available — no pairwise Tanimoto agreement
  term can be formed
- When you need de novo structure elucidation rather than scoring agreement/novelty
  among tool annotations
- When input is a feature table without structural annotations from any of the three
  tools
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MS2DECIDE
  role: Python library whose K_estimation function aggregates GNPS, Sirius, and ISDB-LOTUS
    annotations and applies the Tanimoto-based agreement terms in the knownness score
    k
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Provides one of the three annotation sources (FBMN spectral matching, strict
    or iterative weighted analog search) whose structures enter pairwise Tanimoto
    computation
- name: Sirius
  role: Sirius 6 (Confidence Approximate score) provides structural annotations; missing
    Sirius annotations are assigned matching score 0.5
- name: ISDB-LOTUS
  role: Third annotation source (strict 0.02 Da library search via spectral_lib_matcher/MatchMS);
    zero answers are treated as informative for novelty
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:ffdae45b17958bc23b30dbc80ce3df85bd6abacdbec51043900a2df87ad9e05a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - ISDB-LOTUS
  - MS2DECIDE
  - Sirius
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# Tanimoto similarity thresholding for annotation agreement

## Summary

Computes Tanimoto structural similarity between structural annotations from GNPS, Sirius, and ISDB-LOTUS to activate pairwise agreement terms (phi) in the MS2DECIDE knownness score k, with explicit default rules for missing annotations.

## When to use

Use when aggregating multi-annotations of MS/MS spectra and a pairwise structural-agreement term is needed in the knownness score k, including cases where a tool returned no annotation.

## When NOT to use

- When only a single annotation tool is available — no pairwise Tanimoto agreement term can be formed
- When you need de novo structure elucidation rather than scoring agreement/novelty among tool annotations
- When input is a feature table without structural annotations from any of the three tools

## Inputs

- Structural annotations (InChIs/SMILES) from GNPS FBMN spectral matching (strict or iterative weighted analog search)
- Sirius 6 structure_identifications.tsv annotation file
- ISDB-LOTUS annotations from get_cfm_annotation(mgf, ISDBtol) with mass tolerance <= 0.5 (default 0.02)
- Quantitative feature table (.csv) and MGF file of tandem mass spectra

## Outputs

- Pairwise Tanimoto similarities and phi agreement terms (phi_g,s, phi_g,i, phi_s,i) per annotation pair
- Knownness score k per ion, written to the ranked MS2DECIDE output (.tsv, e.g. k_values.csv for comparison against Table 3)

## How to apply

For each ion feature, compute Tanimoto similarity between the structures (InChIs/SMILES) proposed by each pair of annotation tools (GNPS, Sirius, ISDB-LOTUS). Activate the pairwise agreement term phi_tau,tau0 of Eq. 3 only when the Tanimoto similarity between a pair of annotations is >= 0.5, using the reference phi values from Table 2; this feeds the additive piecewise-linear MAVT aggregation of eta_g, eta_s, and eta_i. Apply the README's missing-annotation defaults: when GNPS or Sirius gives no match and Tanimoto cannot be calculated, assign T_gs = T_gi = 0.7; when Sirius fails to propose an annotation, assign its matching score 0.5. Treat a zero ISDB-LOTUS answer (strict 0.02 Da library search) as informative about novelty rather than discarding the feature. Enforce the tradeoff constraint eta_g(0.8) = eta_s(0.8) + eta_i(0.8) + phi_s,i(0.7, 0.8, 0.8) as a consistency check before composing k.

## Related tools

- **MS2DECIDE** (Python library whose K_estimation function aggregates GNPS, Sirius, and ISDB-LOTUS annotations and applies the Tanimoto-based agreement terms in the knownness score k) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Provides one of the three annotation sources (FBMN spectral matching, strict or iterative weighted analog search) whose structures enter pairwise Tanimoto computation)
- **Sirius** (Sirius 6 (Confidence Approximate score) provides structural annotations; missing Sirius annotations are assigned matching score 0.5)
- **ISDB-LOTUS** (Third annotation source (strict 0.02 Da library search via spectral_lib_matcher/MatchMS); zero answers are treated as informative for novelty)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Agreement terms phi are active only for annotation pairs with Tanimoto >= 0.5; below threshold they contribute zero
- Consistency check: eta_g(0.8) = eta_s(0.8) + eta_i(0.8) + phi_s,i(0.7, 0.8, 0.8) holds within numerical tolerance
- Computed k values per ion match the k column reported in Table 3 within numerical tolerance
- Missing-annotation defaults verified: T_gs = T_gi = 0.7 where GNPS/Sirius matches are absent, and Sirius matching score = 0.5 where Sirius fails
- Final .tsv output is sorted/filtered by K-values as required by K_estimation

## Limitations

- Tanimoto between GNPS and Sirius cannot be calculated when either tool returns no match, forcing the default T_gs = T_gi = 0.7
- The iterative weighted analog GNPS workflow launches 27 FBMN jobs and can take up to three hours; a low library score threshold (0.001) can cause FBMN failures — a higher threshold (e.g. 0.1) is recommended
- Sirius 6 annotation files may contain redundant features in the mappingFeatureId column, causing a 'ValueError: DataFrame index must be unique' error
- ISDB-LOTUS mass tolerance must be < 0.5 (default 0.02 Da)

## Evidence

- [readme] In scenarios where there is no match with GNPS or no match with Sirius, the tanimoto between GNPS and Sirius cannot be calculated. Hence, a default value of 0.7 was assigned to $T_{gs}$ and $T_{gi}$ in these instances.: [evidence span withheld — anchor status: external_doc]
- [readme] Sirius annotations were performed in batch mode by using Sirius 6. we utilized the Confidence Approximate score. Unfortunately, in some cases, Sirius was not able to propose an annotation.: [evidence span withheld — anchor status: external_doc]
- [results] The match-score terms of k are piecewise linear functions broken at 0.4, 0.6, and 0.8, with the greatest slope applied in the interval [0.6, 1.0].: [evidence span withheld — anchor status: not_found]
- [abstract] aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS): "aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS)"
