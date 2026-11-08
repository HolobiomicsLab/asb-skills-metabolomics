---
name: spectral-similarity-scoring
description: Use when an unannotated MS/MS spectrum may be a modified form of a library
  compound — e.g. precursor/fragment shifts consistent with methylation (+14 Da),
  acetylation (+42 Da), hydroxylation (+16 Da), or phosphorylation (+80 Da) — and
  plain library matching failed to assign it.
when_to_use_negative:
- The spectrum already matches a library entry on precursor accurate mass and MS/MS
  — that is MSI level 2 by direct matching, and hybrid search is only for the unannotated
  remainder
- The compound is not plausibly a modified derivative of any library entry (novel
  scaffold) — head-to-tail comparison requires a library parent; use in silico fragmentation
  (CSI:FingerID) instead
- A validated identification confidence level with false-discovery rate assessment
  is required — the field currently lacks such automated calculations, so hybrid-search
  hits remain class-level MSI level 3
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: NIST hybrid search
  role: Hybrid search engine performing the head-to-tail shifted-peak comparison against
    library spectra
- name: NIST17
  role: Mass spectral library supplying the parent-compound spectra that modified
    query spectra are scored against
- name: SIRIUS
  role: LC-MS/MS analysis framework whose demo-data package provides the modified-metabolite
    spectra used to exercise the comparison
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Complementary in silico fragmentation tool for queries with no suitable library
    parent
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:c78ede1f795e891a9aa4b2fdd9c49384f2a485baed2a8d9801e88586ad18f5ed
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - NIST hybrid search
  - NIST17
  - SIRIUS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectral-similarity-scoring

## Summary

Head-to-tail spectral comparison used in the NIST hybrid search, which associates MS/MS spectra of chemically modified metabolites with library entries of their non-modified parent compounds by scoring shifted precursor and fragment mass correspondences (e.g., methyladenosine vs adenosine in NIST17).

## When to use

Use when an unannotated MS/MS spectrum may be a modified form of a library compound — e.g. precursor/fragment shifts consistent with methylation (+14 Da), acetylation (+42 Da), hydroxylation (+16 Da), or phosphorylation (+80 Da) — and plain library matching failed to assign it.

## When NOT to use

- The spectrum already matches a library entry on precursor accurate mass and MS/MS — that is MSI level 2 by direct matching, and hybrid search is only for the unannotated remainder
- The compound is not plausibly a modified derivative of any library entry (novel scaffold) — head-to-tail comparison requires a library parent; use in silico fragmentation (CSI:FingerID) instead
- A validated identification confidence level with false-discovery rate assessment is required — the field currently lacks such automated calculations, so hybrid-search hits remain class-level MSI level 3

## Inputs

- Query MS/MS spectrum (modified metabolite, e.g. methyladenosine) with precursor mass
- NIST17 mass spectral library entries of candidate non-modified parent compounds
- Expected modification mass shifts (+14 methylation, +42 acetylation, +16 hydroxylation, +80 phosphorylation)
- demo-data package from github.com/sirius-ms/sirius containing the four documented modified-metabolite spectra

## Outputs

- Best-scoring parent library entry and match score per query spectrum
- hybrid_search_verification.csv with columns: modified metabolite, precursor shift, fragment shifts, best parent hit, match score
- MSI level 3 structure annotation with associated chemical class

## How to apply

Load the query MS/MS spectrum and candidate parent library spectra (in the paper's demo-data package these are methyladenosine, phosphothreonine, hydroxyarginine, acetylmethionine vs. the adenosine, threonine, arginine, methionine NIST17 library entries). Compute the precursor mass shift relative to each parent and identify fragment mass shifts consistent with the suspected modification mass (+14, +80, +16, +42 Da). Run a head-to-tail alignment that maps modified-spectrum peaks onto parent-library peaks offset by these shifts, scoring each candidate by the number and intensity of shifted peak correspondences. Record the best-scoring parent entry and score per query (e.g., in hybrid_search_verification.csv) and treat the hybrid-search hit as MSI level 3 — a class-level annotation, not a level 1/2 identification.

## Related tools

- **NIST hybrid search** (Hybrid search engine performing the head-to-tail shifted-peak comparison against library spectra)
- **NIST17** (Mass spectral library supplying the parent-compound spectra that modified query spectra are scored against)
- **SIRIUS** (LC-MS/MS analysis framework whose demo-data package provides the modified-metabolite spectra used to exercise the comparison) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Complementary in silico fragmentation tool for queries with no suitable library parent) — https://github.com/sirius-ms/sirius

## Examples

```
import pandas as pd; pd.DataFrame(rows, columns=['modified_metabolite','precursor_shift','fragment_shifts','best_parent_hit','match_score']).to_csv('hybrid_search_verification.csv', index=False)
```

## Evaluation signals

- Each modified spectrum's top-scoring hit is the biologically expected parent (methyladenosine→adenosine, phosphothreonine→threonine, hydroxyarginine→arginine, acetylmethionine→methionine)
- Reported precursor shift for each pair matches the known modification mass (+14, +80, +16, +42 Da respectively)
- Fragment shifts listed for each match are consistent with the single known modification rather than random offsets
- Match scores rank the expected parent above all other library entries scored head-to-tail
- Verification table contains all five required columns and one row per modified metabolite

## Limitations

- Hybrid-search hits are MSI level 3 annotations and many structure assignments require validation by acquiring spectra from corresponding authentic chemicals
- The majority of acquired MS/MS spectra cannot be annotated with certainty to specific chemicals; shifted-peak scoring links compounds but does not prove identity
- No generally accepted and validated automated calculations of identification confidence levels with false-discovery rate assessments exist for metabolomics MS/MS, so hybrid-search scores cannot be converted to FDR-controlled identifications

## Evidence

- [other] spectra of modified metabolites show distinct shifts in precursor masses and fragments when compared against nonmodified library spectra, enabling head-to-tail comparison: [evidence span withheld — anchor status: not_found]
- [readme] SIRIUS is a java-based software framework for the analysis of LC-MS/MS data of metabolites and other "small molecules of biological interest".: [evidence span withheld — anchor status: external_doc]
