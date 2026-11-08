---
name: adduct-and-in-source-fragment-filtering
description: Use when assigning preannotated metabolite candidates (from a reaction-based
  pseudo-MS1 candidature space) to parent pesticides via an MS2 similarity network,
  and adduct/ISF-derived spectral artifacts must be filtered before accepting seed-molecule
  annotations.
when_to_use_negative:
- Candidate spectra lack comparable parent-pesticide MS2 spectra, so no seed-molecule
  similarity link can be built and only pseudo-MS1 (RCS) evidence exists.
- The goal is definitive structural confirmation, e.g. by analytical/stable-isotopic-labeled
  standards; this filter only removes spurious adduct/ISF links, it does not replace
  standard verification.
- The candidate set was not produced by the upstream CMDN RCS layer and lacks accurate
  m/z, RT, and MS2 characterization required for SSN-based assignment.
tools:
- name: CMDN
  role: Framework whose SSN layer hosts the seed-molecule assignment and adduct/ISF
    filtering of candidate-parent links
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: msentropy
  role: R package (dependency installed per the CMDN README) providing entropy-based
    MS2 spectral similarity scoring used to build the SSN
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Data acquisition platform generating the MS2 peak lists of parent pesticides
    and candidate metabolites
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:8f238051ef2bf06ebcff9b0b26a819460894329a97f71b6d83a052999fbd0b54
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  - msentropy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# adduct-and-in-source-fragment-filtering

## Summary

A filtering step in the SSN layer of CMDN that removes candidate-parent pesticide annotations driven by adducts and in-source fragments (ISF), which otherwise generate spurious high-similarity links, and retains only links with entropy-based spectral similarity ss >= 0.4.

## When to use

Use when assigning preannotated metabolite candidates (from a reaction-based pseudo-MS1 candidature space) to parent pesticides via an MS2 similarity network, and adduct/ISF-derived spectral artifacts must be filtered before accepting seed-molecule annotations.

## When NOT to use

- Candidate spectra lack comparable parent-pesticide MS2 spectra, so no seed-molecule similarity link can be built and only pseudo-MS1 (RCS) evidence exists.
- The goal is definitive structural confirmation, e.g. by analytical/stable-isotopic-labeled standards; this filter only removes spurious adduct/ISF links, it does not replace standard verification.
- The candidate set was not produced by the upstream CMDN RCS layer and lacks accurate m/z, RT, and MS2 characterization required for SSN-based assignment.

## Inputs

- MS2 peak lists of parent pesticides (LC-HRMS)
- Preannotated metabolite candidates from the RCS layer with accurate m/z, retention time (RT), and MS2 spectra
- Pairwise entropy-based spectral similarity scores (ss) between candidate and parent MS2 spectra

## Outputs

- ssn_annotations.csv with columns: candidate metabolite, seed parent pesticide, similarity score, filter status

## How to apply

Load the MS2 peak lists of parent pesticides and the preannotated candidate metabolites (accurate m/z, RT, MS2) produced by the upstream reaction-based pseudo-MS1 candidature space (RCS) of CMDN. Compute pairwise entropy-based spectral similarity (ss) between every candidate MS2 spectrum and every parent pesticide MS2 spectrum to build the SSN layer. Assign each candidate the parent with the highest similarity score as its seed molecule, then filter the network by retaining only candidate-parent links with ss >= 0.4 and removing annotations attributable to adducts and in-source fragments (ISF), which otherwise produce spurious high-similarity links. Write surviving confident annotations to ssn_annotations.csv with columns for candidate metabolite, seed parent pesticide, similarity score, and filter status.

## Related tools

- **CMDN** (Framework whose SSN layer hosts the seed-molecule assignment and adduct/ISF filtering of candidate-parent links) — https://github.com/LeaveMeNotTonight/CMDN
- **msentropy** (R package (dependency installed per the CMDN README) providing entropy-based MS2 spectral similarity scoring used to build the SSN) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Data acquisition platform generating the MS2 peak lists of parent pesticides and candidate metabolites)

## Evaluation signals

- Every retained annotation has similarity score ss >= 0.4.
- Each surviving candidate metabolite is assigned exactly one seed molecule (parent pesticide).
- All retained links have passed the adduct/in-source fragment (ISF) removal filter, confirmed via the filter-status column in ssn_annotations.csv.
- Spot-check a subset of seed assignments by expert review against expected biotransformation relationships.
- Schema check: ssn_annotations.csv contains the four expected columns (candidate metabolite, seed parent pesticide, similarity score, filter status) with no spurious adduct/ISF-derived rows.

## Limitations

- Adducts and in-source fragments (ISF) generate spurious high-similarity links in the SSN; without removal, candidates are misassigned to the wrong parent.
- The ss >= 0.4 cutoff is a heuristic threshold; links near the boundary may require manual curation (the article's PMDB annotations of N = 492 underwent manual curation).
- Filtering operates on MS2 spectral similarity and does not by itself validate structures; CMDN relies on additional multichannel validation (standards, RT prediction, time-course logistic modeling, murine studies).

## Evidence

- [methods] Filter the network by retaining only candidate-parent links with ss >= 0.4 and removing annotations attributable to adducts and in-source fragments (ISF): [evidence span withheld — anchor status: not_found]
- [methods] ISF generates spurious high-similarity links: [evidence span withheld — anchor status: not_found]
- [intro] SSN layer within CMDN triple architecture: "(2) a reaction-based pseudo-MS1 candidature space (RCS) and (3) an MS2 spectrum similarity network (SSN)"
- [readme] CMDN framework definition: "Compound metabolite discovery network (CMDN) is an “top-down” untargeted metabolomics-based MS data processing framework to enable high-throughput and automated annotation"
- [readme] Scripts available with raw data: "All the script running can be found from the comment line with raw data provided."
