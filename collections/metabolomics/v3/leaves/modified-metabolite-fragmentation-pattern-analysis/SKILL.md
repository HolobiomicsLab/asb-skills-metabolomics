---
name: modified-metabolite-fragmentation-pattern-analysis
description: Use when annotating MS/MS spectra of suspected modified metabolites that
  fail exact library matching, and you need to associate them with parent-compound
  library entries via mass-shift-consistent fragment correspondences (NIST17 hybrid
  search logic).
when_to_use_negative:
- Spectra already identified at MSI level 1 or 2 with exact precursor mass, retention
  time, and MS/MS matching against authentic standards — direct library matching,
  not hybrid search, is appropriate.
- Compounds lacking any related library entry or suspected parent (e.g., fully novel
  structures) — use in silico fragmentation (CSI:FingerID/SIRIUS) instead, as the
  hybrid search requires a parent library spectrum for head-to-tail comparison.
- MS1-only signals that did not trigger MS/MS fragmentation — no fragment shifts can
  be computed.
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3822
tools:
- name: NIST hybrid search
  role: 'Mechanism being reconstructed: associates modified-metabolite spectra with
    best-scoring related library spectra via shifted peak correspondence'
- name: NIST17
  role: Mass spectral library providing the parent-compound reference spectra compared
    against modified-metabolite spectra
- name: SIRIUS
  role: Framework supplying the demo-data package containing the four modified-metabolite
    MS/MS spectra and parent library entries; complementary in silico fragmentation
    pipeline
  repo: https://github.com/sirius-ms/sirius
- name: CSI:FingerID
  role: Complementary in silico fragmentation tool used alongside the NIST hybrid
    search for MSI level 3 annotation of the same spectra
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:0303376aa9c96b19cf9cda8c84850ab2005792dfb5f3037735b830f231aac167
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
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3822
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# modified-metabolite-fragmentation-pattern-analysis

## Summary

Reconstructs the NIST hybrid search mechanism by comparing MS/MS spectra of chemically modified metabolites (methylated, acetylated, phosphorylated, hydroxylated forms) against library spectra of their non-modified parents, using precursor and fragment mass shifts to link modified compounds to their best-scoring related library entries.

## When to use

Use when annotating MS/MS spectra of suspected modified metabolites that fail exact library matching, and you need to associate them with parent-compound library entries via mass-shift-consistent fragment correspondences (NIST17 hybrid search logic).

## When NOT to use

- Spectra already identified at MSI level 1 or 2 with exact precursor mass, retention time, and MS/MS matching against authentic standards — direct library matching, not hybrid search, is appropriate.
- Compounds lacking any related library entry or suspected parent (e.g., fully novel structures) — use in silico fragmentation (CSI:FingerID/SIRIUS) instead, as the hybrid search requires a parent library spectrum for head-to-tail comparison.
- MS1-only signals that did not trigger MS/MS fragmentation — no fragment shifts can be computed.

## Inputs

- Modified-metabolite MS/MS peak lists from the SIRIUS demo-data package (methyladenosine, phosphothreonine, hydroxyarginine, acetylmethionine)
- Non-modified parent compound library spectra (adenosine, threonine, arginine, methionine)
- Precursor m/z and fragment peak lists (m/z, intensity pairs) for each spectrum

## Outputs

- hybrid_search_verification.csv with columns: modified metabolite, precursor shift, fragment shifts, best parent hit, match score
- Per-pair alignment score based on number and intensity of shifted peak correspondences
- Best-scoring parent library entry for each modified spectrum

## How to apply

Load the modified-metabolite MS/MS spectra (methyladenosine, phosphothreonine, hydroxyarginine, acetylmethionine) and their non-modified parent library entries (adenosine, threonine, arginine, methionine) from the SIRIUS demo-data package. For each pair, compute the precursor mass shift relative to the parent's library spectrum and identify candidate fragment mass shifts consistent with the known modification (+14 Da for methylation, +80 Da for phosphorylation, +16 Da for hydroxylation, +42 Da for acetylation). Implement a head-to-tail spectral comparison that aligns modified-spectrum peaks to parent-library peaks under these shifts, scoring each match by the number and intensity of shifted peak correspondences. Run all four comparisons, record the best-scoring parent entry per modified spectrum, and verify that each modified spectrum resolves to its biologically expected parent as the top hit. This mirrors the NIST hybrid search used in the paper at MSI level 3, where spectra not matching at level 1 (authentic standards with precursor mass, retention time, and MS/MS matching) or level 2 (precursor accurate mass and MS/MS matching alone) are classified by ontology after hybrid search.

## Related tools

- **NIST hybrid search** (Mechanism being reconstructed: associates modified-metabolite spectra with best-scoring related library spectra via shifted peak correspondence)
- **NIST17** (Mass spectral library providing the parent-compound reference spectra compared against modified-metabolite spectra)
- **SIRIUS** (Framework supplying the demo-data package containing the four modified-metabolite MS/MS spectra and parent library entries; complementary in silico fragmentation pipeline) — https://github.com/sirius-ms/sirius
- **CSI:FingerID** (Complementary in silico fragmentation tool used alongside the NIST hybrid search for MSI level 3 annotation of the same spectra) — https://github.com/sirius-ms/sirius

## Evaluation signals

- All four modified spectra (methyladenosine, phosphothreonine, hydroxyarginine, acetylmethionine) resolve to their biologically expected parents (adenosine, threonine, arginine, methionine) as top-scoring hits.
- Computed precursor shifts match known modification masses: +14 Da (methylation), +80 Da (phosphorylation), +16 Da (hydroxylation), +42 Da (acetylation).
- Identified fragment mass shifts are consistent with the same modification magnitude as the precursor shift for each pair.
- hybrid_search_verification.csv contains all required columns (modified metabolite, precursor shift, fragment shifts, best parent hit, match score) with a non-empty score for every row.
- Best-parent scores are non-zero and driven by multiple shifted peak correspondences (number and intensity), not a single fragment match.

## Limitations

- Structure assignments from NIST17 hybrid search require validation with authentic chemicals; many assignments are only 'worthy to be validated'.
- The metabolomics field lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery-rate assessments, so hybrid-search-based MSI level 3 annotations cannot be FDR-calibrated.
- The majority of acquired MS/MS spectra cannot be annotated with certainty to specific chemicals; the hybrid search only links spectra to related library entries, not definitive structures.

## Evidence
