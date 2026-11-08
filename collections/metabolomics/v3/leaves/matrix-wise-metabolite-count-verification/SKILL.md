---
name: matrix-wise-metabolite-count-verification
description: Use when a metabolite-bridging or characterization pipeline reports an
  overall characterization rate and you must confirm per-matrix counts (e.g., 66/222/30/40)
  reconcile with the claimed total and percent improvement.
when_to_use_negative:
- The raw MS feature list or MDRB component relationship network has not yet been
  generated — counts cannot be verified upstream of characterization
- The study involves a single matrix or a herb whose reported per-matrix breakdown
  differs from plasma/urine/feces/lung; the tally structure must be adapted, not copied
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0607
tools:
- name: MDRB
  role: R-based mass dataset relevance bridging pipeline whose output (characterized
    metabolites per matrix) is tallied and verified
  repo: https://github.com/933ZhangDD/MDRB
- name: PATBS
  role: Non-targeted identification step that produces the in vitro prototype list
    from which the characterized metabolite counts derive
- name: R
  role: Language used to run the MDRB data post-processing and compute per-matrix
    characterization rates
  repo: https://github.com/933ZhangDD/MDRB
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13020-024-01031-8
    title: 'An integrated approach for studying exposure, metabolism, and disposition
      of traditional Chinese medicine using PATBS and MDRB tools: a case study of
      semen Armeniacae Amarum'
  source_content_hash: sha256:99b25329fb7732cd5f5b8407f4895cfd91559566ca0c9f8119e4e69e011b25d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/933ZhangDD/MDRB
  tools:
  - MDRB
  - Metworks
  - PATBS
  - R
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0607
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Matrix-wise metabolite count verification

## Summary

Verifies a metabolite characterization campaign by tallying characterized metabolites per biological matrix (plasma, urine, feces, lung) and comparing the totals against reported baseline and improved counts (e.g., 110 at 31% manual vs 227 at 65% MDRB for CKXR).

## When to use

Use when a metabolite-bridging or characterization pipeline reports an overall characterization rate and you must confirm per-matrix counts (e.g., 66/222/30/40) reconcile with the claimed total and percent improvement.

## When NOT to use

- The raw MS feature list or MDRB component relationship network has not yet been generated — counts cannot be verified upstream of characterization
- The study involves a single matrix or a herb whose reported per-matrix breakdown differs from plasma/urine/feces/lung; the tally structure must be adapted, not copied

## Inputs

- MDRB component relationship network output linking in vivo metabolites to in vitro CKXR prototypes, per matrix (plasma, urine, feces, lung)
- Manual characterization baseline count (110 metabolites, 31%)
- Reported target counts: 227 total (65%) with per-matrix 66 plasma / 222 urine / 30 feces / 40 lung

## Outputs

- Per-matrix characterized-metabolite counts (e.g., 66/222/30/40)
- Overall characterization count and rate (e.g., 227 metabolites, 65%)
- Comparison summary table of MDRB vs manual characterization baseline

## How to apply

After running the MDRB pipeline, tally the characterized metabolites linked to in vitro prototypes within each matrix of the component relationship network: plasma, urine, feces, and lung (for CKXR: 66, 222, 30, and 40 respectively). Sum the per-matrix counts and divide by the total metabolite feature list to compute the characterization rate (227/349 = 65%). Construct a summary table with the manual characterization baseline (110 metabolites, 31%) alongside the MDRB result so the improvement is explicitly recorded. Accept the reproduction only when both the total (227 at 65%) and every per-matrix count match the reported values exactly; any mismatch in a single matrix invalidates the verification.

## Related tools

- **MDRB** (R-based mass dataset relevance bridging pipeline whose output (characterized metabolites per matrix) is tallied and verified) — https://github.com/933ZhangDD/MDRB
- **PATBS** (Non-targeted identification step that produces the in vitro prototype list from which the characterized metabolite counts derive)
- **R** (Language used to run the MDRB data post-processing and compute per-matrix characterization rates) — https://github.com/933ZhangDD/MDRB

## Evaluation signals

- Total characterized count equals 227, corresponding to a 65% characterization rate
- Per-matrix counts are exactly 66 (plasma), 222 (urine), 30 (feces), 40 (lung) and sum to the total
- Comparison against the manual baseline of 110 metabolites (31%) is recorded in the summary table
- Every counted metabolite is linked to an in vitro prototype in the component relationship network (no orphan counts)

## Limitations

- Per-matrix and total counts are deterministic outputs of a specific MDRB run; re-running with different scoring thresholds or prototype lists will not reproduce them
- The article reports no statistical validation, replicate counts, or quantitative performance metrics for the characterization strategy
- Identification of in vivo components remains difficult despite MS data post-processing, so counts include characterized (bridged) metabolites, not necessarily fully identified structures

## Evidence

- [abstract] MDRB technology constructed a component relationship network for TCM, significantly improving metabolite identification efficiency: "The MDRB technology constructed a component relationship network for TCM, significantly improving metabolite identification efficiency and facilitating the mapping of translational metabolic pathways."
- [abstract] The current method integrates three dimensions of mass: "The current method integrates three dimensions of mass spectral data including m/z, mass defect of MS1 and MS2, and the similarity of MS2 fragments"
- [intro] MDRB technique was employed to realize the characterization of metabolites and the bridge the correlation between prototype and metabolites: "Second, MDRB technique was employed to realize the characterization of metabolites and the bridge the correlation between prototype and metabolites"
- [discussion] we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes: "we established a technique, MDRB, for in vivo metabolite characterization in TCM and its bridging with in vitro prototypes"
- [discussion] The analysis strategy proposed in this study offers a more efficient and comprehensive approach for the identification and characterization of metabolites of TCMs in vivo: "The analysis strategy proposed in this study offers a more efficient and comprehensive approach for the identification and characterization of metabolites of TCMs in vivo"
