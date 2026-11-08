---
name: ppm-tolerance-mass-search
description: Use when detected m/z values from an SM dataset (e.g. imzML read via
  Cardinal) need metabolite identities without MS/MS, and the platform is not supported
  by high-resolution-only tools like METASPACE or pySM.
when_to_use_negative:
- The data are high-resolution (FT-ICR or orbitrap) and FDR-controlled annotation
  via METASPACE or pySM is available — SpaMTP targets the gap where those tools do
  not apply, including TOF data
- The input is already an annotated feature table rather than raw detected m/z values
- Isomer-level (above MSI level 4) structural resolution is required — exact-mass
  matching reports all possible isomers without discriminating between them
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3336
tools:
- name: SpaMTP
  role: 'R package implementing the annotation engine: BuildMZAnnotationIndex(), QueryMZAnnotationIndex(),
    AdductRules(), MALDIMatrixRules(), AnnotateBigData()'
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: Cardinal
  role: Imports the SM imzML data (e.g. Cardinal::readImzML) from which the m/z peak
    list is derived
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: ms1_id
  role: 'Optional downstream refinement: pseudo-MS/MS evidence from an independent
    indexed GNPS spectral database to support the exact-mass candidates'
  repo: github.com/Philipbear/ms1_id
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:74bf9be4dd642085b74b18b289fe58452c45e7ad0213a25406d7eabb0bd81990
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cardinal
  - R
  - SpaMTP
  - ms1_id
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# ppm-tolerance-mass-search

## Summary

Exact-mass matching of every detected spatial-metabolomics m/z value against pre-compiled compound databases (ChEBI, HMDB, GNPS, LIPIDMAPS SDF-derived) expanded with 47 positive/negative-mode adducts, at a user-defined ppm error (~3 ppm benchmarked), returning formula, InChIKey, adduct type, and all isomer candidates (MSI level 4).

## When to use

Use when detected m/z values from an SM dataset (e.g. imzML read via Cardinal) need metabolite identities without MS/MS, and the platform is not supported by high-resolution-only tools like METASPACE or pySM.

## When NOT to use

- The data are high-resolution (FT-ICR or orbitrap) and FDR-controlled annotation via METASPACE or pySM is available — SpaMTP targets the gap where those tools do not apply, including TOF data
- The input is already an annotated feature table rather than raw detected m/z values
- Isomer-level (above MSI level 4) structural resolution is required — exact-mass matching reports all possible isomers without discriminating between them

## Inputs

- Detected m/z peak list from spatial metabolomics data (e.g. imzML imported with Cardinal::readImzML)
- SDF compound databases: ChEBI, HMDB, GNPS, LIPIDMAPS
- RaMP chem_props reference table (e.g. RAMP_C_000219574 entries)
- User-defined polarity, adduct set, and ppm tolerance (±3 ppm benchmarked)
- Adduct/matrix rules (AdductRules(), MALDIMatrixRules(maldi_matrix, polarity))
- Pre-built annotation index from BuildMZAnnotationIndex()

## Outputs

- Per-peak annotation table (annotation_test_results.csv) with exact mass, formula, annotation name, InChIKey, ppm difference, and adduct type
- All possible isomers per matched m/z (MSI level 4 match)
- Filtered adduct-expanded reference table
- Zenodo deposition artefacts (e.g. results_spotted.csv, annotation_zscore_output.RDS)

## How to apply

Convert the downloaded SDF compound databases (ChEBI, HMDB, GNPS, LIPIDMAPS) into a uniform reference table, removing N/A and zero-value entries. Calculate 47 common adducts in positive and negative mode for each neutral mass entry and append them to the reference table. Filter the database by user-defined polarity, adduct set (rules retrievable via AdductRules(), which returns ion masses, molecular multiplier, charge, proton-loss count, chemical class, and prior; matrix adducts via MALDIMatrixRules()), and an elemental-composition constraint limited to common natural-product elements (C, H, N, S, O, P, Br, F, Na, I, Si). Perform exact-mass matching of every valid m/z against the filtered database at a user-defined ppm error (±3 ppm benchmarked on the human lung data). Because the adduct expansion is expensive, run BuildMZAnnotationIndex() once and query it repeatedly with QueryMZAnnotationIndex(), which uses binary interval search. Report every possible isomer per matched m/z, corresponding to a level 4 Metabolomics Standards Initiative match, and write per-peak results to CSV.

## Related tools

- **SpaMTP** (R package implementing the annotation engine: BuildMZAnnotationIndex(), QueryMZAnnotationIndex(), AdductRules(), MALDIMatrixRules(), AnnotateBigData()) — github.com/GenomicsMachineLearning/SpaMTP
- **Cardinal** (Imports the SM imzML data (e.g. Cardinal::readImzML) from which the m/z peak list is derived) — github.com/GenomicsMachineLearning/SpaMTP
- **ms1_id** (Optional downstream refinement: pseudo-MS/MS evidence from an independent indexed GNPS spectral database to support the exact-mass candidates) — github.com/Philipbear/ms1_id

## Examples

```
fmp10_rules <- MALDIMatrixRules(maldi_matrix = "FMP-10", polarity = "positive")
```

## Evaluation signals

- Every returned annotation must fall within the user-defined ppm error (±3 ppm in the benchmark) — check the reported ppm difference column
- Verify annotation concordance against the deposited reference table results_spotted.csv, consistent with the reported ~90% name agreement with METASPACE (HMDB 87%, LipidMaps 93%)
- Output schema must contain exact mass, formula, annotation name, InChIKey, ppm difference, and adduct type for each match
- All candidate isomers must be reported per matched m/z (level 4 MSI), not a single best hit
- Matched formulae must satisfy the natural-product element constraint (C, H, N, S, O, P, Br, F, Na, I, Si)

## Limitations

- Annotations are MSI level 4 — all possible isomers are reported without isomer discrimination
- No SpaMTP version number is stated in the article; the MS1-ID engine version must be recorded by the user
- The paired targeted metabolomics comparison refinement ('5) Refinement with Paired Targeted Metabolic Data') is a placeholder ('To come!')
- The RaMP-indexed pipeline requires the expensive index build step (BuildMZAnnotationIndex) before querying

## Evidence

- [methods] exact-mass matching of every valid m/z against the filtered database at a user-defined ppm error (±3 ppm as benchmarked on the human lung data): [evidence span withheld — anchor status: not_found]
- [readme] RaMP 3.0 indexed metabolite annotation pipeline: [evidence span withheld — anchor status: external_doc]
