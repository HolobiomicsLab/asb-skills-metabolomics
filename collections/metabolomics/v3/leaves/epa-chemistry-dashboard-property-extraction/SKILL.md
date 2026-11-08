---
name: epa-chemistry-dashboard-property-extraction
description: Use when a chemical list with CASRNs needs toxicity-relevant properties
  (log P, BCF, HTS AC50/efficacy) harvested from the EPA Chemistry Dashboard to feed
  composite prioritization scores such as ToxPi.
when_to_use_negative:
- Chemicals lacking a CASRN — the Dashboard-based HTS retrieval is limited to chemicals
  with CASRN and will fail or return nothing for them
- Analyses needing in vivo or chronic toxicity endpoints — the Dashboard HTS data
  are in vitro assay values and the LD50 approach may underestimate chronic and subchronic
  toxicity
- Compounds outside Dashboard coverage where data retrieval fails entirely (e.g. metals,
  or mixtures needing interaction/synergy assessment)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: U.S. EPA Chemistry Dashboard
  role: Source of ToxCast HTS AC50/efficacy data and log P / BCF property values for
    CASRN-identified chemicals
- name: ToxPI
  role: 'Downstream consumer: computes composite ToxPi scores from the extracted HTS
    and property slices'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1289/ehp7722
    title: Risk-Based Chemical Ranking and Generating a Prioritized Human Exposome
      Database
  source_content_hash: sha256:77c7da8cdc2b634cb053e35d47c4ad5addca7ba0a772ca1e3f708ed6ee838d48
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ToxPI
  - U.S. EPA Chemistry Dashboard
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# epa-chemistry-dashboard-property-extraction

## Summary

Retrieves physicochemical and high-throughput screening data (log P, BCF, ToxCast HTS AC50/efficacy values) from the U.S. EPA Chemistry Dashboard for a chemical list identified by CASRN, to serve as input slices for downstream ToxPi-based toxicity ranking.

## When to use

Use when a chemical list with CASRNs needs toxicity-relevant properties (log P, BCF, HTS AC50/efficacy) harvested from the EPA Chemistry Dashboard to feed composite prioritization scores such as ToxPi.

## When NOT to use

- Chemicals lacking a CASRN — the Dashboard-based HTS retrieval is limited to chemicals with CASRN and will fail or return nothing for them
- Analyses needing in vivo or chronic toxicity endpoints — the Dashboard HTS data are in vitro assay values and the LD50 approach may underestimate chronic and subchronic toxicity
- Compounds outside Dashboard coverage where data retrieval fails entirely (e.g. metals, or mixtures needing interaction/synergy assessment)

## Inputs

- Chemical list with CASRN identifiers (from the HExpMetDB / human exposome database of >20,000 chemicals)
- U.S. EPA Chemistry Dashboard queries returning ToxCast HTS AC50 and efficacy data (97 assays)
- Dashboard-retrieved log P and BCF physicochemical property values

## Outputs

- Per-chemical property/HTS table mapping CASRN to 97 HTS assay AC50/efficacy values, log P, and BCF
- Feature table formatted for ToxPi GUI 2.0 composite toxicity slices, feeding ranked toxpi_scores.csv with 95% confidence intervals

## How to apply

Start from a curated chemical list where each entry carries a CASRN, because the Dashboard lookup only succeeds for chemicals with valid CASRN identifiers. Query the U.S. EPA Chemistry Dashboard per chemical to pull the ToxCast AC50 and efficacy data across the 97 in vitro HTS assays used in this study, plus log P and BCF values needed for bioaccumulation/exposure slices. Merge the retrieved HTS hits and property values into a single chemical-by-feature table (8,845 chemicals in the reference workflow), mapping assay endpoints and physicochemical parameters into distinct toxicity slices for ToxPi GUI 2.0. The output table (e.g. toxpi_scores.csv inputs) should retain CASRN, assay identifiers, AC50/efficacy values, log P, and BCF so that composite ToxPi scores and bootstrap 95% confidence intervals can be computed and chemicals ranked (score range 0.00–0.50 in the reference run).

## Related tools

- **U.S. EPA Chemistry Dashboard** (Source of ToxCast HTS AC50/efficacy data and log P / BCF property values for CASRN-identified chemicals)
- **ToxPI** (Downstream consumer: computes composite ToxPi scores from the extracted HTS and property slices)

## Evaluation signals

- Property retrieval succeeds for exactly the CASRN-bearing subset of the chemical list (8,845 chemicals with HTS data in the reference workflow)
- Retrieved HTS data cover 97 ToxCast in vitro assays with AC50 and efficacy values populated per chemical
- Downstream ToxPi scores span 0.00–0.50 across all chemicals with valid bootstrap 95% CIs
- Top-ranked chemicals match the reference run: 17α-ethinylestradiol and tributyltin chloride at 0.50 (95% CIs 0.22–0.62 and 0.26–0.67)
- Output table retains CASRN, assay identifiers, log P, and BCF for every row so scores are reproducible

## Limitations

- HTS retrieval from the U.S. EPA Chemistry Dashboard works only for chemicals with CASRN; chemicals without CASRN cannot be assigned domain data or ToxPi scores
- Toxicity ranking based on these HTS/property slices may underestimate risk of chronic and subchronic toxicity, and cannot exclude mixture interaction (e.g. synergistic) effects
- External-exposure-based scores omit IVIVE; external-to-internal exposure conversion is needed for more physiologically meaningful prioritization

## Evidence

- [discussion] Dashboard-limited CASRN retrieval: "our HTS method is based on the U.S. EPA Chemistry Dashboard, only chemicals with CASRN were successful in retrieving diverse types of relevant domain data"
- [discussion] ToxPi score computation: "we further calculated the ToxPi score in order to more comprehensively estimate the toxicity ranking of chemicals"
- [abstract] ToxCast as HTS data source: "the Toxicological Priority Index calculated from the ToxCast database"
- [abstract] Integrated prioritization workflow: "chemicals were prioritized by integrating exposure level obtained from the Systematic Empirical Evaluation of Models with toxicity data predicted by the Toxicity Estimation Software Tool and the"
