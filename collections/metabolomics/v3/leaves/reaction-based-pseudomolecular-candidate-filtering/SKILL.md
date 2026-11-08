---
name: reaction-based-pseudomolecular-candidate-filtering
description: Use when LC-HRMS feature tables from S9 incubation or cohort samples
  contain a known parent xenobiotic (e.g., a pesticide) and you must enumerate reaction-derived
  metabolite candidates via mass-shift rules before MS2 spectral-network annotation.
when_to_use_negative:
- The goal is bottom-up discovery against spectral reference libraries for intact
  (non-biotransformed) chemicals — RCS requires a known parent to seed mass-shift
  candidates
- Input lacks accurate-mass feature data (e.g., low-resolution MS only), since pseudo-MS1
  candidature depends on m/z matching of reaction-rule mass deltas
- Authentic analytical or stable-isotopic-labeled standards are already available
  and direct standard verification is planned instead of candidate enumeration
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: CMDN
  role: Framework implementing the reaction-based pseudo-MS1 candidature space (RCS);
    R scripts in the LeaveMeNotTonight/CMDN repo apply the reaction rules to parent
    m/z values
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Data acquisition platform providing the accurate m/z, retention time, and
    MS2 spectra that form the input feature table for candidate generation
- name: MSDial
  role: Compatible software (ver. 4.80 per README) used to produce the deconvoluted
    feature table that feeds the RCS candidate space
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:e5f7bd3782c36c2d62b2145c4d817d664d06b26517eb2b2da12f84891459ada3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  - MSDial
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# reaction-based-pseudomolecular-candidate-filtering

## Summary

The reaction-based pseudo-MS1 candidature space (RCS) of CMDN proposes candidate xenobiotic metabolites 'top-down' by applying biotransformation reaction rules (mass shifts such as demethylation, +O, S-to-O substitution) to a known parent compound's LC-HRMS feature data, instead of relying on spectral reference libraries.

## When to use

Use when LC-HRMS feature tables from S9 incubation or cohort samples contain a known parent xenobiotic (e.g., a pesticide) and you must enumerate reaction-derived metabolite candidates via mass-shift rules before MS2 spectral-network annotation.

## When NOT to use

- The goal is bottom-up discovery against spectral reference libraries for intact (non-biotransformed) chemicals — RCS requires a known parent to seed mass-shift candidates
- Input lacks accurate-mass feature data (e.g., low-resolution MS only), since pseudo-MS1 candidature depends on m/z matching of reaction-rule mass deltas
- Authentic analytical or stable-isotopic-labeled standards are already available and direct standard verification is planned instead of candidate enumeration

## Inputs

- LC-HRMS feature table (accurate m/z, retention time, MS2 spectra) from parent-compound incubation samples (CSV/xlsx readable with pandas or R readxl)
- Known parent compound identifier and exact m/z (e.g., malathion, sulprofos)
- CMDN reaction-rule set (290 biotransformation rules, supplementary Table)

## Outputs

- Candidate metabolite list with m/z, RT shift, and biotransformation type per parent (e.g., 12 metabolites for the malathion subnetwork, 4 for sulprofos)
- Annotated candidates formatted for downstream CMDN SSN spectral-similarity network construction
- Subnetwork tables with node metadata and edge similarity scores (e.g., representative_subnetworks.csv)

## How to apply

Load the LC-HRMS feature table containing accurate m/z, retention time (RT), and MS2 spectra for incubation samples (e.g., batch 1 malathion and sulprofos S9 incubations) with pandas or R tidyverse. Apply CMDN's reaction rules (290 rules per the supplementary tables) to the parent's exact m/z to generate candidate metabolites linked by observed biotransformations — for malathion: demethylation (−CH2), sulfur-to-oxygen substitution (−S+O), +O, and the two-step −CH2–S+O; for sulprofos: −S+O, +O, and −S+O+O. Each candidate is a pseudo-MS1 hit: an m/z matching the parent plus the rule's mass delta, with no reference library lookup, which is the rationale for discovering metabolites absent from spectra databases. Annotate each candidate with its RT shift and biotransformation type, then pass the annotated candidates to the MS2 spectrum similarity network (SSN) module (CluMSID/MSMSsim/msentropy pairwise entropy similarities) for subnetwork extraction. The reaction-rule set used for CMDN's curated PMDB annotations (N = 492 with manual curation) is provided in the repository's supplementary tables.

## Related tools

- **CMDN** (Framework implementing the reaction-based pseudo-MS1 candidature space (RCS); R scripts in the LeaveMeNotTonight/CMDN repo apply the reaction rules to parent m/z values) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Data acquisition platform providing the accurate m/z, retention time, and MS2 spectra that form the input feature table for candidate generation)
- **MSDial** (Compatible software (ver. 4.80 per README) used to produce the deconvoluted feature table that feeds the RCS candidate space)

## Examples

```
import pandas as pd; ft = pd.read_csv('batch1_LCHRMS_feature_table.csv'); candidates = apply_reaction_rules(ft, parent='malathion', rules=['-CH2','-S+O','+O','-CH2-S+O'])
```

## Evaluation signals

- Every proposed candidate's m/z equals the parent m/z plus one of the rule mass deltas (e.g., −CH2 for demethylation, −S+O substitution) within the instrument's ppm tolerance
- Each candidate node carries a biotransformation-type annotation and an RT-shift value traceable to a rule in the 290-rule set
- No candidate is generated without a valid parent–rule linkage (candidates are 'top-down' from parents, never library matches)
- Downstream check: SSN built from RCS candidates reproduces the reported subnetwork sizes — exactly 12 metabolite rows for malathion and 4 for sulprofos in the output table
- Manually curated CMDN-annotated PMDB entries (N = 492) serve as the reference set for spot-checking annotation correctness

## Limitations

- Requires a known parent compound and pre-defined reaction rules; in silico prediction and fragmentation simulation platforms 'often fall short of providing direct evidence under real-world, omics-scale conditions'
- Annotated candidates need validation through multiple channels (standards, RT prediction, time-course logistic modeling, murine studies); PMDB entries required manual curation
- Scarcity of authentic standards for reaction-derived metabolites limits direct confirmation of RCS-proposed candidates

## Evidence
