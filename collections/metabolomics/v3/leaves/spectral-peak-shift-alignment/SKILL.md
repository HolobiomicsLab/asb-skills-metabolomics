---
name: spectral-peak-shift-alignment
description: Use when an MS/MS spectrum of a putatively modified metabolite (e.g.
  methylated, acetylated, phosphorylated, hydroxylated) fails direct library search
  and you need to associate it with the -scoring parent library entry via consistent
  mass shifts.
when_to_use_negative:
- The goal is MSI level 1 identification using authentic standards with precursor
  mass, retention time, and MS/MS matching — direct matching, not shift-based hybrid
  search, is the appropriate route
- The query spectrum already matches a library entry directly at MSI level 2 by precursor
  accurate mass and MS/MS matching alone — hybrid search is only needed for otherwise
  unannotated spectra
- No library spectrum of a plausible non-modified parent compound exists, so no head-to-tail
  shifted comparison can be anchored
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: NIST hybrid search
  role: 'Provides the hybrid search mechanism being reconstructed: head-to-tail comparison
    of a modified query spectrum against non-modified library spectra using precursor
    and fragment mass shifts'
- name: NIST17
  role: Mass spectral library supplying the non-modified parent compound entries against
    which modified spectra are compared
- name: SIRIUS
  role: Java framework for LC-MS/MS analysis whose demo-data package supplies the
    modified-metabolite and parent-compound spectra used to exercise the alignment
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:1015b2eb121a6949917c021483009ab9ee14448f80dbe20d5fe54e6830ddca0f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - NIST hybrid search
  - NIST17
  - SIRIUS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spectral-peak-shift-alignment

## Summary

Reimplements the NIST17 hybrid search mechanism, which links MS/MS spectra of chemically modified metabolites to the library spectra of their non-modified parent compounds by aligning peaks under a common precursor and fragment mass shift (head-to-tail comparison). This supports MSI level 3 structure annotation of spectra that fail direct library matching.

## When to use

Use when an MS/MS spectrum of a putatively modified metabolite (e.g. methylated, acetylated, phosphorylated, hydroxylated) fails direct library search and you need to associate it with the best-scoring parent library entry via consistent mass shifts.

## When NOT to use

- The goal is MSI level 1 identification using authentic standards with precursor mass, retention time, and MS/MS matching — direct matching, not shift-based hybrid search, is the appropriate route
- The query spectrum already matches a library entry directly at MSI level 2 by precursor accurate mass and MS/MS matching alone — hybrid search is only needed for otherwise unannotated spectra
- No library spectrum of a plausible non-modified parent compound exists, so no head-to-tail shifted comparison can be anchored

## Inputs

- MS/MS spectra of modified metabolites (methyladenosine, phosphothreonine, hydroxyarginine, acetylmethionine) from the SIRIUS demo-data package
- Mass spectral library entries (NIST17) of the corresponding non-modified parent compounds (adenosine, threonine, arginine, methionine)
- Expected modification mass shifts (+14 Da methylation, +80 Da phosphorylation, +16 Da hydroxylation, +42 Da acetylation)

## Outputs

- hybrid_search_verification.csv with columns: modified metabolite, precursor shift, fragment shifts, best parent hit, match score
- Ranked list of best-scoring parent library entries per modified spectrum

## How to apply

Load the modified-metabolite MS/MS spectra and their candidate non-modified parent library entries from the SIRIUS demo-data package (e.g. methyladenosine vs adenosine, phosphothreonine vs threonine, hydroxyarginine vs arginine, acetylmethionine vs methionine). For each pair, compute the precursor mass shift relative to the parent library spectrum and identify candidate fragment mass shifts consistent with the known modification (+14 Da methylation, +80 Da phosphorylation, +16 Da hydroxylation, +42 Da acetylation). Implement the head-to-tail spectral comparison: align modified-spectrum peaks to parent-library peaks offset by these shifts, scoring each match by the number and intensity of shifted peak correspondences. Rank all candidate parent entries per modified spectrum, record the best-scoring hit and its score, and verify that each modified spectrum retrieves its biologically expected parent as the top hit. Write the verification table to hybrid_search_verification.csv with columns for modified metabolite, precursor shift, fragment shifts, best parent hit, and match score.

## Related tools

- **NIST hybrid search** (Provides the hybrid search mechanism being reconstructed: head-to-tail comparison of a modified query spectrum against non-modified library spectra using precursor and fragment mass shifts)
- **NIST17** (Mass spectral library supplying the non-modified parent compound entries against which modified spectra are compared)
- **SIRIUS** (Java framework for LC-MS/MS analysis whose demo-data package supplies the modified-metabolite and parent-compound spectra used to exercise the alignment) — https://github.com/sirius-ms/sirius

## Examples

```
pairs = {'methyladenosine':('adenosine',+14.0),'phosphothreonine':('threonine',+80.0),'hydroxyarginine':('arginine',+16.0),'acetylmethionine':('methionine',+42.0)}; # compute precursor/fragment shifts, head-to-tail align, write hybrid_search_verification.csv
```

## Evaluation signals

- Each of the four modified spectra (methyladenosine, phosphothreonine, hydroxyarginine, acetylmethionine) retrieves its biologically expected parent (adenosine, threonine, arginine, methionine) as the top-scoring match
- Computed precursor shifts equal the expected modification masses: +14 Da (methylation), +80 Da (phosphorylation), +16 Da (hydroxylation), +42 Da (acetylation)
- Match scores reflect both the number and the intensity of shifted peak correspondences, not just peak counts
- hybrid_search_verification.csv exists with all five required columns populated for all four pairs
- Fragment mass shifts are internally consistent with the same modification mass as the precursor shift for each pair

## Limitations

- Hybrid-search-derived associations are MSI level 3 annotations: many structure assignments still need to be validated by acquiring spectra from corresponding authentic chemicals
- The majority of acquired MS/MS spectra cannot be annotated with certainty to specific chemicals, so correct parent retrieval on a small demo set does not generalize to arbitrary unknowns
- The field of metabolomics currently lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments, so shift-based scores have no calibrated FDR interpretation

## Evidence

- [readme] SIRIUS is the LC-MS/MS analysis framework used in this workflow: [evidence span withheld — anchor status: external_doc]
