---
name: intra-subclass-unsaturation-parallelism-checking
description: Decision support for ranking candidate lipid annotations in LC-MS/MS
  data when multiple isobaric chain-composition assignments exist within one subclass
  and only retention order can discriminate them.
when_to_use_negative:
- The input is an unannotated feature table with no lipid subclass labels or chain
  compositions — the rule can only be checked after MS/MS-based annotation has assigned
  subclasses and double-bond counts.
- Retention times come from direct infusion, GC, or SFC-MS rather than reversed-phase
  LC-MS/MS lipidomics, where the intra-subclass RT ordering underlying the rule does
  not apply.
- The goal is absolute RT prediction (e.g. via the WMYn model) rather than relative-order
  conformity checking between candidate annotations.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3520
tools:
- name: LipidIN
  role: Provides the subclass/chain-composition enumeration (theoretical library)
    and the LCI scoring that consumes the parallelism conformity result; its Python
    pkl libraries define which subclasses and unsaturation patterns to group.
  repo: github.com/LinShuhaiLAB/LipidIN
- name: Python
  role: Implementation language used to group features by subclass, bin by double-bond
    count, compute pairwise RT-order conformity, and write conformity_summary.csv.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:fcc29c933d1656c8d2b8c30e29da2d3f7ea7a33e2c8fa99eb3fea5db115ad464
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - Python
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# intra-subclass-unsaturation-parallelism-checking

## Summary

Verifies the second of LipidIN's three relative retention-time rules: within a lipid subclass, lipids sharing the same number of C=C double bonds must elute in parallel RT order as acyl chain length varies. It is the empirical basis for the LC categories intelligence (LCI) module's RT-based annotation confidence.

## When to use

Decision support for ranking candidate lipid annotations in LC-MS/MS data when multiple isobaric chain-composition assignments exist within one subclass and only retention order can discriminate them.

## When NOT to use

- The input is an unannotated feature table with no lipid subclass labels or chain compositions — the rule can only be checked after MS/MS-based annotation has assigned subclasses and double-bond counts.
- Retention times come from direct infusion, GC, or SFC-MS rather than reversed-phase LC-MS/MS lipidomics, where the intra-subclass RT ordering underlying the rule does not apply.
- The goal is absolute RT prediction (e.g. via the WMYn model) rather than relative-order conformity checking between candidate annotations.

## Inputs

- Annotated lipid feature table (lipid identity, subclass, chain composition, double-bond count, observed retention time) from LC-MS/MS runs
- Published datasets referenced in Fig. 2 and Supplementary Data 2 that provide annotated lipid identities with retention times
- LipidIN Python theoretical-library .pkl files (pos_common.pkl / neg_common.pkl, 24,631,702 spectra) to enumerate subclass chain compositions

## Outputs

- Per-dataset intra-subclass parallelism conformity fractions (fraction of lipid pairs whose observed RT order conforms to the parallelism rule)
- Aggregated median conformity across datasets (expected ≈90%)
- Flagged RT-discordant annotations to be down-weighted in LCI scoring
- conformity_summary.csv with per-dataset and aggregated metrics

## How to apply

1) Group annotated features by lipid subclass (e.g. PC, TG, PE as parsed by the LipidIN theoretical library's 168.6-million hierarchical entries) and record their observed RTs from the LC run. 2) Within each subclass, bin lipids by double-bond count (degree of unsaturation of each fatty acyl) and verify that, for equal double-bond counts, RT ordering is parallel across differing total acyl carbon numbers — i.e. a series with more carbons elutes consistently later/earlier in the same pattern as its unsaturated analogues. 3) Flag any intra-subclass pair whose observed RT order contradicts the parallelism expectation as an RT-discordant candidate and down-weight or reject that annotation. 4) Feed the resulting conformity fraction into the LCI scoring used to combine MS/MS- and RT-based evidence, which the paper reports yields a 5.7% estimated false discovery rate across 8,923 lipids. 5) Validate by reproducing the conformity statistics: run the check over the >100 published datasets referenced in Fig. 2/Supplementary Data 2 and confirm median conformity ≈90%. Rationale: the paper hypothesizes that deep learning models can extract the mapping between lipid structural features (head group, chain length, degree of unsaturation) and retention time, and that combining MS/MS- and RT-based scores improves annotation performance in LC-MS/MS lipidomics; the parallelism rule is the second of three RT rules operationalizing that mapping.

## Related tools

- **LipidIN** (Provides the subclass/chain-composition enumeration (theoretical library) and the LCI scoring that consumes the parallelism conformity result; its Python pkl libraries define which subclasses and unsaturation patterns to group.) — github.com/LinShuhaiLAB/LipidIN
- **Python** (Implementation language used to group features by subclass, bin by double-bond count, compute pairwise RT-order conformity, and write conformity_summary.csv.)

## Examples

```
python scripts/check_rt_parallelism.py --annotated features_annotated.csv --library pos_common.pkl --out conformity_summary.csv
```

## Evaluation signals

- Median conformity across >100 datasets reproduces the reported ~90% value.
- Average absolute RT deviation rate for the ECN rule reproduces 1.44% and for the ESCN rule 1.28% when the same datasets are re-scored with the sibling rules (co-consistency check).
- Every flagged RT-discordant pair has both members within the same subclass and equal double-bond counts, and the contradiction is a genuine ordering reversal rather than an RT tolerance artifact.
- Applying the rule within LCI scoring yields the reported 5.7% estimated false discovery rate across 8,923 lipids.
- conformity_summary.csv contains one row per dataset plus aggregated rows with conformity fraction and deviation-rate columns.

## Limitations

- Median conformity is ~90%, not 100% — roughly one in ten lipid pairs violate the parallelism rule even in published datasets, so RT-discordant pairs are down-weighted rather than absolutely rejected.
- The rule depends on annotated lipid identities and retention times being available; the deeper relationship between metabolites and relative RT remains uninvestigated per the discussion section.
- Conformity statistics depend on the specific set of published datasets analyzed; different LC methods or columns may shift the observed conformity fraction.
- The article pre-resolves the sibling ECN rule deviation (1.44%) and ESCN rule deviation (1.28%) but the parallelism rule's own per-pair deviation threshold is not stated numerically in the provided context, so implementers must define an RT tolerance before scoring.

## Evidence

- [abstract] Parallelism rule is rule 2 of 3 in LCI: "developed using three relative retention time rules, reducing false positive annotations and predicting unannotated lipids with a 5.7% estimated false discovery rate, covering 8923 lipids cross"
- [intro] Rule derived from structural-feature-to-RT mapping: "We hypothesized that deep learning models could extract the mapping relationship between lipid structure features (e.g., head group, chain length, and degree of unsaturation of each fatty acyl or"
- [other] Validation target: ~90% conformity across datasets: [evidence span withheld — anchor status: not_found]
- [other] Workflow step defining the grouping criterion: [evidence span withheld — anchor status: not_found]
- [discussion] LCI module basis: "Future research directions include pre-training on a larger and more diverse dataset to extend reverse lipidomics."
