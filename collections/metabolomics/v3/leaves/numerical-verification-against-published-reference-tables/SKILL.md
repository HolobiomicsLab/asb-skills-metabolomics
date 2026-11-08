---
name: numerical-verification-against-published-reference-tables
description: Use when a paper reports per-item numerical outputs (e.g. per-ion knownness
  scores in Table 3) alongside explicit functional forms and parameter tables, and
  the goal is to independently reproduce and validate those numbers.
when_to_use_negative:
- The paper provides only final scores with no functional form, breakpoints, or parameter
  tables to reconstruct the model from
- The target is running MS2DECIDE on new data for prioritization rather than verifying
  the published numbers
tools:
- name: MS2DECIDE
  role: Published implementation whose K_estimation output can be cross-checked against
    the reconstructed k values
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Source of GNPS match-score annotations whose pairwise Tanimoto similarities
    feed the phi agreement terms
- name: Sirius
  role: Source of Sirius match-score annotations (default 0.5 when absent) used in
    the k aggregation
- name: ISDB-LOTUS
  role: Source of ISDB-LOTUS annotations; zero answers are treated as informative
    for novelty
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:58edd3b6edf439fffe7a3466040cf89865acfd81ba3312c5b5d568f41eb14954
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
yes_rate: 0.875
n_claims: 8
---

# numerical-verification-against-published-reference-tables

## Summary

Re-implements a published quantitative model (here, the MS2DECIDE knownness score k built from piecewise-linear MAVT aggregation, Eq. 2-3 with Table 1-2 parameters) and checks the recomputed values against the k column reported in the paper's reference table (Table 3) within a numerical tolerance.

## When to use

Use when a paper reports per-item numerical outputs (e.g. per-ion knownness scores in Table 3) alongside explicit functional forms and parameter tables, and the goal is to independently reproduce and validate those numbers.

## When NOT to use

- The paper provides only final scores with no functional form, breakpoints, or parameter tables to reconstruct the model from
- The target is running MS2DECIDE on new data for prioritization rather than verifying the published numbers

## Inputs

- Piecewise-linear component definitions and expert reference values (Table 1)
- Pairwise agreement (phi) reference values (Table 2)
- Per-ion annotation pairs and published k values (Table 3)
- Optional: MS2DECIDE inputs (quantitative .csv, MGF file, Sirius 6 structure_identifications.tsv) for end-to-end recomputation

## Outputs

- k_values.csv with computed k per ion
- Mismatch report flagging ions whose recomputed k differs from Table 3 beyond numerical tolerance

## How to apply

Encode each component function (eta_g, eta_s, eta_i) as a piecewise-linear function of the normalized score tau with breakpoints at 0.4, 0.6, 0.8, and 1.0, using the expert reference values from Table 1, giving the greatest slope to the interval [0.6, 1.0] where match-score differences matter most to the expert. Implement the pairwise agreement term phi_tau,tau0 of Eq. 3, activating it only when the Tanimoto similarity between a pair of annotations is >= 0.5, with reference phi values from Table 2; handle missing-tool cases with the README defaults (T_gs = T_gi = 0.7 when GNPS-Sirius Tanimoto cannot be computed; Sirius matching score = 0.5 when no annotation is proposed). Enforce the tradeoff constraint eta_g(0.8) = eta_s(0.8) + eta_i(0.8) + phi_s,i(0.7, 0.8, 0.8) as a consistency check on the component functions before composing k. Evaluate the composed k on the annotation pairs of the ions listed in Table 3, write the computed values to k_values.csv, and flag any mismatch versus the published k column beyond numerical tolerance.

## Related tools

- **MS2DECIDE** (Published implementation whose K_estimation output can be cross-checked against the reconstructed k values) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Source of GNPS match-score annotations whose pairwise Tanimoto similarities feed the phi agreement terms)
- **Sirius** (Source of Sirius match-score annotations (default 0.5 when absent) used in the k aggregation)
- **ISDB-LOTUS** (Source of ISDB-LOTUS annotations; zero answers are treated as informative for novelty)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- Tradeoff identity eta_g(0.8) = eta_s(0.8) + eta_i(0.8) + phi_s,i(0.7, 0.8, 0.8) holds for the encoded component functions
- Piecewise-linear segments break exactly at tau = 0.4, 0.6, 0.8, 1.0 with the steepest slope on [0.6, 1.0]
- phi terms are zero (inactive) for annotation pairs with Tanimoto similarity < 0.5
- Recomputed k values in k_values.csv match the Table 3 k column within numerical tolerance for every ion in Table 3

## Limitations

- Default substitutions when a tool fails to annotate (T_gs = T_gi = 0.7; Sirius score = 0.5) mean reconstructed k can differ from a run with complete annotations
- Redundant features in Sirius structure_identifications.tsv (duplicate mappingFeatureId) raise 'DataFrame index must be unique for orient=\'index\'' errors during recomputation
- Only comparisons within tolerance can be verified; the README notes failures can occur with the FBMN GNPS workflow at the 0.001 score threshold, recommending an increased threshold (e.g. 0.1)

## Evidence

- [other] The match-score terms of k are piecewise linear functions broken at 0.4, 0.6, and 0.8, with the greatest slope applied in the interval [0.6, 1.0]: [evidence span withheld — anchor status: not_found]
- [readme] In scenarios where there is no match with GNPS or no match with Sirius, the tanimoto between GNPS and Sirius cannot be calculated. Hence, a default value of 0.7 was assigned: [evidence span withheld — anchor status: external_doc]
- [readme] in some cases, Sirius was not able to propose an annotation. To remedy, we associated a value of 0.5 to Sirius matching score.: [evidence span withheld — anchor status: external_doc]
- [abstract] MS2DECIDE reliably captures the novelty of natural products from their tandem mass spectra, as shown in two case studies: [evidence span withheld — anchor status: not_found]
