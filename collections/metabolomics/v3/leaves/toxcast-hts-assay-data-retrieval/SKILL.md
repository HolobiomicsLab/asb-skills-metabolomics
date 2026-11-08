---
name: toxcast-hts-assay-data-retrieval
description: Use when chemicals carry CASRNs and you need in vitro HTS assay activity
  (AC50/efficacy) from ToxCast plus physicochemical data to compute ToxPi scores for
  risk-based chemical prioritization.
when_to_use_negative:
- Chemicals lack CASRNs — the Dashboard-based HTS retrieval fails for them (only CASRN-bearing
  chemicals succeeded in this study)
- 'You need chronic/subchronic toxicity estimates: the HTS/ToxPi and LD50-based ranking
  may underestimate chronic and subchronic toxicity risk'
- You need internal-dose risk scores — the paper notes the risk score used external
  exposure and IVIVE modeling was not yet incorporated
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0203
tools:
- name: ToxPI
  role: Computes composite ToxPi scores by integrating HTS assay data, log P, and
    BCF into toxicity slices for ranking
- name: U.S. EPA Chemistry Dashboard
  role: Source of ToxCast HTS data (AC50, efficacy) and log P / BCF values, keyed
    by CASRN
- name: TEST
  role: Supplies predicted oral LD50 values for rats used alongside ToxPi in the toxicity
    ranking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:e9ccf5bfe7d4f6149e35955fccd92eda510230e04a11ecf2843a96342a4353f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TEST
  - ToxPI
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# toxcast-hts-assay-data-retrieval

## Summary

Retrieves ToxCast high-throughput screening (HTS) assay data via the U.S. EPA Chemistry Dashboard for chemicals identified by CASRN, and integrates these HTS data with log P and BCF values into composite ToxPi toxicity scores for chemical ranking.

## When to use

Use when chemicals carry CASRNs and you need in vitro HTS assay activity (AC50/efficacy) from ToxCast plus physicochemical data to compute ToxPi scores for risk-based chemical prioritization.

## When NOT to use

- Chemicals lack CASRNs — the Dashboard-based HTS retrieval fails for them (only CASRN-bearing chemicals succeeded in this study)
- You need chronic/subchronic toxicity estimates: the HTS/ToxPi and LD50-based ranking may underestimate chronic and subchronic toxicity risk
- You need internal-dose risk scores — the paper notes the risk score used external exposure and IVIVE modeling was not yet incorporated

## Inputs

- Chemical list with CASRNs (e.g., human exposome chemicals from HExpMetDB)
- ToxCast HTS assay data (AC50, efficacy) retrieved via the U.S. EPA Chemistry Dashboard
- log P and BCF values from the U.S. EPA Chemistry Dashboard

## Outputs

- Ranked table of ToxPi scores with bootstrap 95% confidence intervals per chemical (toxpi_scores.csv)
- Filter chemical subset with HTS data (e.g., 8,845 of the >20,000 exposome chemicals)

## How to apply

Start from a chemical list with valid CASRNs — only chemicals with CASRN successfully retrieve data from the U.S. EPA Chemistry Dashboard. Retrieve ToxCast AC50 and efficacy data for the relevant HTS assays (e.g., 97 in vitro assays in this study) plus log P and BCF values from the Dashboard, then filter the chemical list to those with HTS data (8,845 chemicals here). Compute ToxPi scores using ToxPi GUI 2.0 by integrating the HTS assay data, log P, and BCF into composite toxicity slices. Rank chemicals by ToxPi score and bootstrap 95% confidence intervals for each score. This HTS-based scoring complements the TEST-predicted oral LD50 values used in the paper's risk-index prioritization.

## Related tools

- **ToxPI** (Computes composite ToxPi scores by integrating HTS assay data, log P, and BCF into toxicity slices for ranking)
- **U.S. EPA Chemistry Dashboard** (Source of ToxCast HTS data (AC50, efficacy) and log P / BCF values, keyed by CASRN)
- **TEST** (Supplies predicted oral LD50 values for rats used alongside ToxPi in the toxicity ranking)

## Evaluation signals

- Exactly 8,845 chemicals receive ToxPi scores after filtering to those with HTS data
- Score range spans 0.00–0.50
- Top-ranked chemicals are 17α-ethinylestradiol and tributyltin chloride at 0.50, with 95% CIs 0.22–0.62 and 0.26–0.67 respectively
- Every scored chemical has a valid CASRN (Dashboard retrieval prerequisite)
- Output CSV includes per-chemical scores with bootstrapped 95% confidence intervals

## Limitations

- Only chemicals with CASRN could retrieve diverse types of relevant domain data from the U.S. EPA Chemistry Dashboard
- The risk score was calculated using external exposure; IVIVE modeling was not included and should be considered in future studies
- LD50/HTS-based ranking may underestimate the risk of chronic and subchronic toxicity, and mixture effects from chemical interactions (e.g., synergistic effects) cannot be excluded

## Evidence

- [discussion] our HTS method is based on the U.S. EPA Chemistry Dashboard, only chemicals with CASRN were successful in retrieving diverse types of relevant domain data: "our HTS method is based on the U.S. EPA Chemistry Dashboard, only chemicals with CASRN were successful in retrieving diverse types of relevant domain data"
- [discussion] we further calculated the ToxPi score in order to more comprehensively estimate the toxicity ranking of chemicals: "we further calculated the ToxPi score in order to more comprehensively estimate the toxicity ranking of chemicals"
- [abstract] chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models with toxicity data predicted by the Toxicity Estimation Software Tool: "chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models with toxicity data predicted by the Toxicity Estimation Software Tool"
- [abstract] the Toxicological Priority Index calculated from the ToxCast database: "the Toxicological Priority Index calculated from the ToxCast database"
- [discussion] may underestimate the risk of the chemicals for their chronic and subchronic toxicity: "may underestimate the risk of the chemicals for their chronic and subchronic toxicity"
