---
name: decision-theory-aggregation-function-implementation
description: 'Use when reconstructing or re-implementing the MS2DECIDE knownness score
  k: multi-annotated MS/MS features from GNPS, Sirius, and ISDB-LOTUS must be ranked
  by an expert-modeled, piecewise-linear preference function.'
when_to_use_negative:
- Input consists of only one annotation source (no GNPS/Sirius/ISDB-LOTUS multiannotation),
  since pairwise agreement terms phi require cross-tool Tanimoto comparisons
- The goal is a generic machine-learning scoring model rather than reproducing the
  expert-elicited MAVT preference structure with fixed Table 1/Table 2 parameters
tools:
- name: MS2DECIDE
  role: Python library implementing the K_estimation aggregation of GNPS, ISDB-LOTUS,
    and Sirius annotations into knownness K-values
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: One of the three annotation sources aggregated by k; FBMN spectral matching
    (strict or iterative weighted analog search) provides match scores
- name: Sirius
  role: Annotation source run in batch mode with Sirius 6 Confidence Approximate score;
    supplies match scores and mappingFeatureId links to features
- name: ISDB-LOTUS
  role: Annotation source matched via spectral_lib_matcher (MatchMS) against ISDB-LOTUS
    spectra with 0.02 Da strict mass tolerance
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:475874a182a38beb160311eb0b1a313dc5d29d56090653adfd18676ec33e006a
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# decision-theory-aggregation-function-implementation

## Summary

Implements the knownness score k of MS2DECIDE as an additive piecewise-linear MAVT aggregation (Eq. 2-3, Tables 1-2) of GNPS, Sirius, and ISDB-LOTUS match scores, encoding expert-elicited preferences into piecewise-linear eta and pairwise-agreement phi terms.

## When to use

Use when reconstructing or re-implementing the MS2DECIDE knownness score k: multi-annotated MS/MS features from GNPS, Sirius, and ISDB-LOTUS must be ranked by an expert-modeled, piecewise-linear preference function.

## When NOT to use

- Input consists of only one annotation source (no GNPS/Sirius/ISDB-LOTUS multiannotation), since pairwise agreement terms phi require cross-tool Tanimoto comparisons
- The goal is a generic machine-learning scoring model rather than reproducing the expert-elicited MAVT preference structure with fixed Table 1/Table 2 parameters

## Inputs

- Quantitative feature table (.csv, MZmine FBMN export >2.53 format)
- Tandem mass spectrometry data in MGF format (.mgf)
- Sirius 6 annotation file (structure_identifications.tsv)
- Reference parameter tables: Table 1 (eta breakpoints/values) and Table 2 (phi agreement values)
- Table 3 ions with reported k values for validation

## Outputs

- k_values.csv with computed knownness scores per ion for comparison against Table 3
- Ranked .tsv file of annotations filtered and sorted by K-values (K_estimation output)

## How to apply

Encode eta_g, eta_s, and eta_i as piecewise-linear functions of the normalized match score tau with breakpoints at 0.4, 0.6, 0.8, and 1.0, using the reference values of Table 1; the greatest slope is deliberately placed in the interval [0.6, 1.0] because the expert considers match-score differences most important there. Implement the pairwise agreement term phi_tau,tau0 of Eq. 3, activating it only when the Tanimoto similarity between two annotations is >= 0.5, with reference phi values from Table 2, and enforce the consistency constraint eta_g(0.8) = eta_s(0.8) + eta_i(0.8) + phi_s,i(0.7, 0.8, 0.8). Compose k as the aggregation of the three eta terms and the phi agreement terms, then evaluate it on the annotation pairs of the Table 3 ions and compare computed values against the reported k column beyond numerical tolerance. Handle missing annotations as the README prescribes: default T_gs and T_gi to 0.7 when the GNPS-Sirius Tanimoto cannot be computed, assign a 0.5 Sirius matching score when Sirius proposes no annotation, and treat a zero ISDB-LOTUS answer (strict 0.02 Da search) as meaningful information about novelty.

## Related tools

- **MS2DECIDE** (Python library implementing the K_estimation aggregation of GNPS, ISDB-LOTUS, and Sirius annotations into knownness K-values) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (One of the three annotation sources aggregated by k; FBMN spectral matching (strict or iterative weighted analog search) provides match scores)
- **Sirius** (Annotation source run in batch mode with Sirius 6 Confidence Approximate score; supplies match scores and mappingFeatureId links to features)
- **ISDB-LOTUS** (Annotation source matched via spectral_lib_matcher (MatchMS) against ISDB-LOTUS spectra with 0.02 Da strict mass tolerance)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Consistency check passes: eta_g(0.8) = eta_s(0.8) + eta_i(0.8) + phi_s,i(0.7, 0.8, 0.8)
- Computed k values per ion (k_values.csv) match the k column of Table 3 within numerical tolerance
- Piecewise-linear eta functions change slope exactly at breakpoints 0.4, 0.6, 0.8, 1.0 with the steepest slope on [0.6, 1.0]
- phi agreement terms are inactive for annotation pairs with Tanimoto similarity < 0.5
- Output .tsv is filtered and sorted by K-values, and default-value substitutions (T_gs = T_gi = 0.7, Sirius score 0.5) only occur where expected

## Limitations

- When there is no GNPS or no Sirius match, the GNPS-Sirius Tanimoto cannot be calculated, so a default value of 0.7 is assigned to T_gs and T_gi
- Sirius sometimes fails to propose an annotation; a value of 0.5 is then associated to the Sirius matching score
- The iterative weighted GNPS analog search launches 27 FBMN jobs and can take up to three hours; a library score threshold of 0.001 can cause FBMN failures, so increasing it (e.g., 0.1) is recommended
- Redundant features in structure_identifications.tsv (mappingFeatureId column) raise 'ValueError: DataFrame index must be unique for orient=index' and must be removed

## Evidence

- [abstract] aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS): [evidence span withheld — anchor status: not_found]
- [readme] piecewise-linear breakpoints rationale: [evidence span withheld — anchor status: external_doc]
- [readme] default Tanimoto when no match: [evidence span withheld — anchor status: external_doc]
- [readme] Sirius missing annotation handling: [evidence span withheld — anchor status: external_doc]
- [readme] ISDB-LOTUS zero answer: [evidence span withheld — anchor status: external_doc]
