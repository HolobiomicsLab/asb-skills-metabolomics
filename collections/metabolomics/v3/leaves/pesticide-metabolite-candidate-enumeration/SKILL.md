---
name: pesticide-metabolite-candidate-enumeration
description: Use when you have a library of parent xenobiotic structures and LC-HRMS
  feature lists from treated vs. control samples, and need to enumerate reaction-rule-derived
  metabolite candidates and match their theoretical m/z to observed dysregulated features
  without reference standards.
when_to_use_negative:
- When observed features already have confident MS2 reference-library or standard-based
  matches — the paper's motivation is that traditional 'bottom-up' workflows relying
  on reference libraries restrict discovery of previously unknown metabolites, so
  rule-based enumeration is unnecessary overhead for already-annotated features
- When the compound class of interest has no representative biotransformation rules
  in the curated rule set (the 290 rules are phase-I rules oriented to pesticide/xenobiotic
  enzymatic metabolism)
- When only low-resolution MS data are available — the 5 ppm mass-error criterion
  requires high-resolution accurate-mass LC-HRMS features
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: CMDN
  role: Framework whose RCS layer performs reaction-rule-driven candidate enumeration
    and 5 ppm preannotation against DMS features; scripts run in R
  repo: github.com/LeaveMeNotTonight/CMDN
- name: BioTransformer 3.0
  role: Compatible in silico biotransformation prediction tool that can supply metabolite
    candidates/masses
- name: MS-Dial (ver. 4.80)
  role: Compatible processing software for producing the feature lists (m/z, RT) that
    feed the DMS layer used in matching
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:72b255e803d1924ea9e5aa53a55505b2bd409dbd06bcfb165f3a9923ba593340
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BioTransformer 3.0
  - CMDN
  - MS-Dial (ver. 4.80)
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pesticide-metabolite-candidate-enumeration

## Summary

Enumerates theoretical biotransformed metabolite candidates by applying curated reaction rules in silico to a parent chemical library (e.g., 1,021 pesticides against 290 nonisomeric phase-I KEGG-derived rules), forming a pseudo-MS1 candidature space (RCS) that is preannotated against dysregulated features by accurate-mass matching at ≤ 5 ppm.

## When to use

Use when you have a library of parent xenobiotic structures and LC-HRMS feature lists from treated vs. control samples, and need to enumerate reaction-rule-derived metabolite candidates and match their theoretical m/z to observed dysregulated features without reference standards.

## When NOT to use

- When observed features already have confident MS2 reference-library or standard-based matches — the paper's motivation is that traditional 'bottom-up' workflows relying on reference libraries restrict discovery of previously unknown metabolites, so rule-based enumeration is unnecessary overhead for already-annotated features
- When the compound class of interest has no representative biotransformation rules in the curated rule set (the 290 rules are phase-I rules oriented to pesticide/xenobiotic enzymatic metabolism)
- When only low-resolution MS data are available — the 5 ppm mass-error criterion requires high-resolution accurate-mass LC-HRMS features

## Inputs

- Parent chemical structure library (e.g., 1,021 pesticides used in CMDN)
- Curated nonisomeric biotransformation reaction rules (290 phase-I rules from KEGG and curated literature, shipped with CMDN)
- DMS-layer output: table of dysregulated features from differential LC-HRMS analysis with accurate m/z values

## Outputs

- Nonredundant set of theoretical biotransformed candidates with monoisotopic m/z values (pseudo-MS1 candidature space)
- rcs_candidate_features.csv: per preannotated feature — theoretical m/z, parent pesticide identity, applied reaction rule, and m/z error in ppm

## How to apply

Load the parent structure library (in the CMDN paper, 1,021 pesticides) and the curated nonisomeric phase-I reaction rules (290 rules sourced from KEGG and literature, as listed in the CMDN supplementary materials). Apply each rule in silico to every parent compound to enumerate theoretical biotransformed derivatives and compute their monoisotopic m/z values, deduplicating to a nonredundant pseudo-MS1 candidature space. Separately, obtain the dysregulated features flagged by the differential metabolite space (DMS) layer from the LC-HRMS data. Cross-reference theoretical candidate m/z values against DMS features using a mass-error tolerance of 5 ppm to preannotate candidates. Emit a per-feature table (rcs_candidate_features.csv) recording theoretical m/z, parent identity, applied reaction rule, and m/z error in ppm; the downstream MS2 similarity network (SSN) layer then resolves final annotations.

## Related tools

- **CMDN** (Framework whose RCS layer performs reaction-rule-driven candidate enumeration and 5 ppm preannotation against DMS features; scripts run in R) — github.com/LeaveMeNotTonight/CMDN
- **BioTransformer 3.0** (Compatible in silico biotransformation prediction tool that can supply metabolite candidates/masses)
- **MS-Dial (ver. 4.80)** (Compatible processing software for producing the feature lists (m/z, RT) that feed the DMS layer used in matching)

## Evaluation signals

- Every cross-referenced candidate–feature pair satisfies the 5 ppm mass-error criterion (m/z error in ppm recorded per entry and all ≤ 5)
- The enumerated nonredundant candidate set is consistent in scale with the reported outcome: 2,886 nonredundant biotransformed derivatives from 1,021 pesticides
- Each row of rcs_candidate_features.csv carries the four required fields: theoretical m/z, parent pesticide identity, applied reaction rule, and m/z error in ppm
- Reaction-rule set matches the curated CMDN rule count (n = 290 nonisomeric phase-I rules) and is traceable to KEGG/literature sources
- All preannotated candidates trace back to a parent compound in the input 1,021-compound library (no orphan candidates)

## Limitations

- Rule-based enumeration generates a candidate space that still requires downstream disambiguation — the MS2 spectrum similarity network (SSN) layer and multichannel validation (standard verification, RT prediction, murine studies, time-course logistic modeling) are needed to confirm annotations
- Candidates are theoretical pseudo-MS1 entries; m/z matching alone provides no structural or isomeric resolution beyond the reaction rule applied
- Rule coverage is limited to the 290 curated nonisomeric phase-I rules, so metabolites formed by reactions outside this set (e.g., unmodeled phase-II conjugation or non-enzymatic pathways) will be missed

## Evidence
