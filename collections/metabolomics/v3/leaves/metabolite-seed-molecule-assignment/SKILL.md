---
name: metabolite-seed-molecule-assignment
description: Use when preannotated metabolite candidates (accurate m/z, RT, MS2) from
  a reaction-based pseudo-MS1 candidature step must be linked back to their parent
  pesticides via MS2 spectral similarity.
when_to_use_negative:
- Candidates lack MS2 spectra (pseudo-MS1-only features), since the SSN layer requires
  pairwise MS2 spectral comparison
- No reference MS2 spectra of parent compounds are available, so no seed-molecule
  similarity scores can be computed
- The goal is discovering de novo metabolites from raw spectra; the RCS candidature
  step must run first to supply the candidate space
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: CMDN
  role: Framework whose SSN layer performs the entropy-based seed-molecule assignment
    of candidates to parent pesticides
  repo: LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Acquires the accurate m/z, RT, and MS2 spectra of parents and candidates that
    feed the SSN similarity computation
- name: msentropy
  role: R package (listed in the CMDN README) used to compute the entropy-based MS2
    spectral similarity scores
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:3685764acfc9d437b77255d083988fc147172a2f464dc1c4149211e7aa97666e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  - msentropy
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolite-seed-molecule-assignment

## Summary

Assigns preannotated xenobiotic metabolite candidates to their parent compounds within the SSN layer of the CMDN framework by designating, for each candidate MS2 spectrum, the parent pesticide with the highest entropy-based spectral similarity score as its "seed molecule."

## When to use

Use when preannotated metabolite candidates (accurate m/z, RT, MS2) from a reaction-based pseudo-MS1 candidature step must be linked back to their parent pesticides via MS2 spectral similarity.

## When NOT to use

- Candidates lack MS2 spectra (pseudo-MS1-only features), since the SSN layer requires pairwise MS2 spectral comparison
- No reference MS2 spectra of parent compounds are available, so no seed-molecule similarity scores can be computed
- The goal is discovering de novo metabolites from raw spectra; the RCS candidature step must run first to supply the candidate space

## Inputs

- MS2 peak lists of parent pesticides (e.g., the 1,021 pesticides in the enzymatic S9 biotransformation assays)
- Preannotated metabolite candidates from the reaction-based pseudo-MS1 candidature space (RCS), each with accurate m/z, RT, and MS2 spectrum

## Outputs

- ssn_annotations.csv with columns for candidate metabolite, seed parent pesticide, similarity score, and filter status
- Filtered MS2 spectrum similarity network (SSN) retaining only candidate-parent links with ss >= 0.4 and adduct/ISF links removed

## How to apply

Load the MS2 peak lists of parent pesticides and the preannotated candidate metabolites produced by the upstream reaction-based pseudo-MS1 candidature space (RCS) of CMDN, each characterized by accurate m/z, RT, and MS2 spectra. Compute pairwise entropy-based spectral similarity (ss) between every candidate MS2 spectrum and every parent pesticide MS2 spectrum to build the SSN layer, using the R msentropy package installed alongside CluMSID and OrgMassSpecR in the CMDN pipeline. Assign each candidate the parent pesticide with the highest similarity score as its seed molecule, then filter the network by retaining only candidate-parent links with ss >= 0.4. Remove spurious high-similarity links attributable to adducts and in-source fragments (ISF), which would otherwise inflate similarity scores. Write surviving confident annotations to ssn_annotations.csv with columns for candidate metabolite, seed parent pesticide, similarity score, and filter status.

## Related tools

- **CMDN** (Framework whose SSN layer performs the entropy-based seed-molecule assignment of candidates to parent pesticides) — LeaveMeNotTonight/CMDN
- **LC-HRMS** (Acquires the accurate m/z, RT, and MS2 spectra of parents and candidates that feed the SSN similarity computation)
- **msentropy** (R package (listed in the CMDN README) used to compute the entropy-based MS2 spectral similarity scores)

## Evaluation signals

- Every retained annotation has ss >= 0.4
- Each candidate metabolite is assigned exactly one seed molecule (the parent with the highest similarity score)
- All annotations have passed the adduct/in-source fragment (ISF) removal filter; no spurious adduct- or ISF-derived links remain
- ssn_annotations.csv contains the expected columns: candidate metabolite, seed parent pesticide, similarity score, and filter status
- A spot-checked subset of seed assignments is confirmed by expert review

## Limitations

- Adducts and in-source fragments (ISF) can generate spurious high-similarity links and must be explicitly filtered or they contaminate seed assignments
- Assignments depend on the quality of the upstream reaction-based pseudo-MS1 candidature space; CMDN annotations in the PMDB still required manual curation (N = 492 curated entries)
- Manual curation was needed even after automated annotation, indicating the SSN layer alone is not fully authoritative

## Evidence
