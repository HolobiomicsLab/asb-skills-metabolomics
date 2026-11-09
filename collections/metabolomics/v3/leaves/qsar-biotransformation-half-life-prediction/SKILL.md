---
name: qsar-biotransformation-half-life-prediction
description: Use when human biotransformation half-lives (HLBs) must be estimated
  for hundreds-to-thousands of organic chemicals that have log Kow/log Koa values
  but no measured HLB, e.g. for persistence screening or exposome risk prioritization.
when_to_use_negative:
- The chemical contains a metal atom or has molecular weight > 1,000 — the IFS approach
  cannot predict HLBs for these chemicals
- The goal is predicting biotransformation metabolite products rather than half-lives
  — that is handled by the BioTransformer machine-learning method in the same pipeline
- Measured biotransformation half-life data already exist for the chemicals; use measured
  values instead of QSAR predictions
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Iterative Fragment Selection QSAR (Arnot et al. 2014)
  role: 'Core prediction engine: uses log Kow/log Koa fragment-based descriptors to
    estimate human biotransformation half-lives for each chemical'
- name: UFZ-LSER database v3.2.1
  role: Source of the log Kow and log Koa partition-coefficient descriptors required
    as IFS input
- name: BioTransformer
  role: Companion tool in the same pipeline for predicting biotransformation metabolite
    products; HLB estimation itself comes from IFS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:356298da39e1fe71c4cd20ac6cac84e0496198e972a22b6658c2299012bcbcd1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioTransformer
  - Iterative Fragment Selection QSAR (Arnot et al. 2014)
  - UFZ-LSER database v3.2.1
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# qsar-biotransformation-half-life-prediction

## Summary

Predicts human biotransformation half-lives (HLBs) for large chemical inventories using the Iterative Fragment Selection (IFS) QSAR approach (Arnot et al. 2014), driven by log Kow and log Koa partition-coefficient descriptors. Used to flag rapidly biotransformed chemicals (<12 h) when building a risk-prioritized human exposome database.

## When to use

Use when human biotransformation half-lives (HLBs) must be estimated for hundreds-to-thousands of organic chemicals that have log Kow/log Koa values but no measured HLB, e.g. for persistence screening or exposome risk prioritization.

## When NOT to use

- The chemical contains a metal atom or has molecular weight > 1,000 — the IFS approach cannot predict HLBs for these chemicals
- The goal is predicting biotransformation metabolite products rather than half-lives — that is handled by the BioTransformer machine-learning method in the same pipeline
- Measured biotransformation half-life data already exist for the chemicals; use measured values instead of QSAR predictions

## Inputs

- Compiled chemical list with identifiers (e.g. CASRN) for exposome chemicals
- log Kow and log Koa values per chemical retrieved from UFZ-LSER database v3.2.1
- Molecular weight and elemental composition (to apply the metal-atom / MW > 1,000 exclusion filter)

## Outputs

- Table of chemical identifiers with predicted human biotransformation half-lives (HLBs) in hours
- Summary statistics: count of successfully predicted chemicals, median HLB, and fraction with HLB < 12 h

## How to apply

Start from the compiled chemical list and retrieve log Kow and log Koa for each chemical from the UFZ-LSER database v3.2.1. Filter out chemicals containing a metal atom or with molecular weight > 1,000, which the IFS approach cannot handle. Apply the Iterative Fragment Selection QSAR (Arnot et al. 2014) using the retrieved partition-coefficient descriptors to predict human biotransformation half-lives. Assemble predicted HLB values into a table keyed by chemical identifiers (CASRN), then summarize the fraction with HLB < 12 h to identify rapidly biotransformed substances. Validate against reference values for well-studied chemicals (DEHP 4.5 h, BPA 7.1 h, PFOA 7,786 h, PCB-105 21,809 h) before accepting the run.

## Related tools

- **Iterative Fragment Selection QSAR (Arnot et al. 2014)** (Core prediction engine: uses log Kow/log Koa fragment-based descriptors to estimate human biotransformation half-lives for each chemical)
- **UFZ-LSER database v3.2.1** (Source of the log Kow and log Koa partition-coefficient descriptors required as IFS input)
- **BioTransformer** (Companion tool in the same pipeline for predicting biotransformation metabolite products; HLB estimation itself comes from IFS)

## Evaluation signals

- HLBs are successfully predicted for 19,406 chemicals in the compiled human exposome database
- Median predicted HLB equals 4.96 h
- Roughly 70% of substances (13,733/19,406) have predicted half-lives < 12 h
- Spot-check values match reported half-lives: DEHP 4.5 h, BPA 7.1 h, PFOA 7,786 h, PCB-105 21,809 h
- No output chemical violates the exclusion filter (i.e. no metal-containing or MW > 1,000 chemical carries a predicted HLB)

## Limitations

- The IFS approach cannot predict HLBs for chemicals with a metal atom or molecular weight > 1,000
- Predictions are QSAR estimates; BioTransformer's companion metabolite predictions can be incomplete or inaccurate for some chemicals
- Coverage depends on descriptor availability — chemicals without retrievable log Kow/log Koa from UFZ-LSER v3.2.1 cannot be processed

## Evidence

- [abstract] The biotransformation half-lives (HLBs) of all the chemicals were assessed using the Iterative Fragment Selection approach: "The biotransformation half-lives (HLBs) of all the chemicals were assessed using the Iterative Fragment Selection approach"
- [discussion] Metal/MW exclusion: "we cannot predict the HLBs of chemicals with a metal atom or molecular weight of >1,000 using the IFS approach"
- [discussion] BioTransformer caveat: "BioTransformer cannot predict all metabolites, and the prediction results may also be inaccurate for some chemicals"
