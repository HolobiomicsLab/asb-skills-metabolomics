---
name: spectral-similarity-network-construction
description: Use when MS2 peak lists exist for both known parent compounds (e.g.,
  pesticides) and preannotated candidate metabolites (accurate m/z, RT, MS2) from
  an upstream reaction-based pseudo-MS1 candidature space, and candidates must be
  assigned to parents by spectral similarity.
when_to_use_negative:
- When performing traditional 'bottom-up' identification against reference spectral
  libraries, which CMDN explicitly avoids because it restricts discovery of previously
  unknown xenobiotic metabolites
- When no MS2 spectra are available for the parent compounds — the seed-molecule assignment
  requires pairwise candidate-parent spectral comparison
- When the candidate space has not been filtered for adducts and in-source fragments
  (ISF), since these generate spurious high-similarity links that corrupt the network
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CMDN
  role: Overall framework; the SSN is its third layer, following the differential
    metabolite space (DMS) and reaction-based pseudo-MS1 candidature space (RCS)
  repo: github.com/LeaveMeNotTonight/CMDN
- name: msentropy
  role: R package used to compute entropy-based pairwise MS2 spectral similarity scores
    (ss) between candidate and parent spectra
- name: CluMSID
  role: R dependency of the CMDN scripts for MS2 similarity clustering in the spectral
    networking workflow
- name: LC-HRMS
  role: Instrument platform producing the accurate m/z, RT, and MS2 spectra that feed
    the SSN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:b67bf977ba84c30afdd98a718dcf448bf8732cff9f09ee72f6754f4b7f5fb4e4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CMDN
  - CluMSID
  - LC-HRMS
  - msentropy
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# spectral-similarity-network-construction

## Summary

Constructs an MS2 spectrum similarity network (SSN) — the third layer of the CMDN framework — that links preannotated metabolite candidates to parent pesticides via entropy-based MS2 spectral similarity, designating the highest-scoring parent as the 'seed molecule'.

## When to use

Use when MS2 peak lists exist for both known parent compounds (e.g., pesticides) and preannotated candidate metabolites (accurate m/z, RT, MS2) from an upstream reaction-based pseudo-MS1 candidature space, and candidates must be assigned to parents by spectral similarity.

## When NOT to use

- When performing traditional 'bottom-up' identification against reference spectral libraries, which CMDN explicitly avoids because it restricts discovery of previously unknown xenobiotic metabolites
- When no MS2 spectra are available for the parent compounds — the seed-molecule assignment requires pairwise candidate-parent spectral comparison
- When the candidate space has not been filtered for adducts and in-source fragments (ISF), since these generate spurious high-similarity links that corrupt the network

## Inputs

- MS2 peak lists of parent pesticide compounds (from LC-HRMS acquisition)
- Preannotated metabolite candidate spectra with accurate m/z, retention time (RT), and MS2 spectra from the reaction-based pseudo-MS1 candidature space (RCS)

## Outputs

- ssn_annotations.csv with columns for candidate metabolite, seed parent pesticide, similarity score, and filter status
- MS2 spectrum similarity network (SSN) of candidate-parent links with entropy similarity scores

## How to apply

Load MS2 peak lists of parent pesticides and candidate metabolites produced by the upstream reaction-based pseudo-MS1 candidature space (RCS) of CMDN. Compute pairwise entropy-based spectral similarity (ss) between every candidate MS2 spectrum and every parent pesticide MS2 spectrum using the msentropy R package (part of the CMDN R dependency stack alongside CluMSID, MSMSsim, and OrgMassSpecR) to build the SSN. Assign each candidate the parent with the highest similarity score as its seed molecule. Retain only candidate-parent links with ss >= 0.4, and remove annotations attributable to adducts and in-source fragments (ISF), which otherwise generate spurious high-similarity links. The rationale is that reaction-derived metabolites share fragmentation patterns with their parents, so high entropy similarity provides direct spectral evidence without reference spectral libraries.

## Related tools

- **CMDN** (Overall framework; the SSN is its third layer, following the differential metabolite space (DMS) and reaction-based pseudo-MS1 candidature space (RCS)) — github.com/LeaveMeNotTonight/CMDN
- **msentropy** (R package used to compute entropy-based pairwise MS2 spectral similarity scores (ss) between candidate and parent spectra)
- **CluMSID** (R dependency of the CMDN scripts for MS2 similarity clustering in the spectral networking workflow)
- **LC-HRMS** (Instrument platform producing the accurate m/z, RT, and MS2 spectra that feed the SSN)

## Evaluation signals

- Every retained annotation in ssn_annotations.csv has similarity score ss >= 0.4
- Each candidate metabolite is assigned exactly one seed molecule (the parent with the highest spectral similarity)
- All surviving annotations have passed the adduct/in-source fragment (ISF) removal filter
- The output table contains the required columns: candidate metabolite, seed parent pesticide, similarity score, and filter status
- A subset of seed assignments is spot-checked by expert review for correctness

## Limitations

- Adducts and in-source fragments (ISF) can generate spurious high-similarity links and must be explicitly removed or they will corrupt the network
- The ss >= 0.4 cutoff may exclude metabolites whose biotransformation substantially alters the fragmentation pattern
- Requires reference MS2 spectra for the parent compounds; performance depends on the quality of the upstream RCS candidate space and on manual curation (e.g., the CMDN-annotated PMDB with manual curation, N = 492)

## Evidence

- [other] In the SSN layer, the parent compound with the highest spectral similarity score is designated as the "seed molecule," and the corresponding candidate is annotated as its most likely metabolite.: [evidence span withheld — anchor status: not_found]
