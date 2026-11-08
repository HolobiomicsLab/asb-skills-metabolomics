---
name: spectral-library-pair-eligibility-filtering
description: Use when building an analog-pair benchmark from GNPS MS/MS spectral libraries
  (e.g., GNPS-MSMLS, NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, BERKELEY-LAB) and
  you need to keep only structurally related pairs with exactly one modification site.
when_to_use_negative:
- Your data are experimental sample spectra rather than GNPS library compound pairs
  — the eligibility checks (identical M+H adduct, single modification site) are defined
  for curated library analogs
- You permit multiple modification sites between the known and unknown compounds —
  the RDKit GetSubstructMatch edge check requires exactly one
- You are refining substructure annotations within a single pair (e.g., MF-R helper
  exclusion of compounds within 0.5 precursor mass, or MF-O oracle filtering) — that
  is downstream substructure filtering, not pair-level eligibility screening
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ModiFinder
  role: Downstream tool that consumes the filtered pair set for modification-site
    localization and Average-distance evaluation (MF-N/MF-R/MF-O)
  repo: github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Combinatorial fragmentation (breadth-first, max depth 2) annotating peaks
    with substructures at 40 ppm so shifted-peak explainability can be tested during
    filtering
- name: RDKit
  role: GetSubstructMatch used to count structural differences and verify exactly
    one modification site per pair
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:8485a515ff0273583f247a29de509387ee009e26890da9b47b539e3eb3f46064
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MAGMa
  - MS/MS library search
  - ModiFinder
  - RDKit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spectral-library-pair-eligibility-filtering

## Summary

Filters GNPS MS/MS library compound pairs down to structurally valid analog pairs suitable for ModiFinder modification-site evaluation, applying eligibility checks (precursor mass < 2000 Da, mass difference < 50%, identical M+H adduct, exactly one modification site) before running MF-N/MF-R/MF-O scoring.

## When to use

Use when building an analog-pair benchmark from GNPS MS/MS spectral libraries (e.g., GNPS-MSMLS, NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, BERKELEY-LAB) and you need to keep only structurally related pairs with exactly one modification site.

## When NOT to use

- Your data are experimental sample spectra rather than GNPS library compound pairs — the eligibility checks (identical M+H adduct, single modification site) are defined for curated library analogs
- You permit multiple modification sites between the known and unknown compounds — the RDKit GetSubstructMatch edge check requires exactly one
- You are refining substructure annotations within a single pair (e.g., MF-R helper exclusion of compounds within 0.5 precursor mass, or MF-O oracle filtering) — that is downstream substructure filtering, not pair-level eligibility screening

## Inputs

- GNPS MS/MS spectral library entries from GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, and BERKELEY-LAB
- Known/unknown compound structures (for RDKit GetSubstructMatch) with precursor masses and adduct labels (M+H)
- MS2-known and MS2-unknown peak lists (MGF-style spectra) for shifted/unshifted peak alignment at 40 ppm

## Outputs

- Filtered benchmark pair set (8033 pairs, 62% of candidates) with at least one shifted peak explainable by a substructure of the parent compound
- Per-pair eligibility flags for the four checks (precursor < 2000 Da, <50% mass difference, identical M+H adduct, exactly one modification site)
- Eligible pairs suitable for MF-N, MF-R, and MF-O Average-distance evaluation

## How to apply

Build candidate pairs from the GNPS library datasets and apply four eligibility checks: (i) precursor mass < 2000 Da, (ii) precursor mass difference < 50% of the smaller compound's mass, (iii) identical M+H adduct, and (iv) exactly one modification site, verified with RDKit GetSubstructMatch edge counting between the known and unknown structures. Retain only pairs where at least one shifted peak is explainable by a substructure of the parent compound (annotated via MAGMa combinatorial fragmentation at depth 2, 40 ppm tolerance) after removing peaks below 1% of the base peak intensity and normalizing to a Euclidean norm of 1. This filtering keeps the majority of pairs (62% of the total, 8033 pairs) for evaluating ModiFinder variants. Note that helper-compound refinement (MF-R) additionally excludes any compound within 0.5 precursor mass of the unknown, but that is a separate downstream filter, not the eligibility screen.

## Related tools

- **ModiFinder** (Downstream tool that consumes the filtered pair set for modification-site localization and Average-distance evaluation (MF-N/MF-R/MF-O)) — github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Combinatorial fragmentation (breadth-first, max depth 2) annotating peaks with substructures at 40 ppm so shifted-peak explainability can be tested during filtering)
- **RDKit** (GetSubstructMatch used to count structural differences and verify exactly one modification site per pair)

## Examples

```
from rdkit import Chem; keep = (mass_known < 2000 and precursor_diff < 0.5*min(mass_known, mass_unknown) and Chem.MolFromSmiles(s_known).HasSubstructMatch(Chem.MolFromSmiles(s_unknown)))
```

## Evaluation signals

- Kept-pair fraction matches the reported majority retention: 62% of the total, i.e. 8033 pairs remain after filtering
- Every retained pair passes all four checks: precursor < 2000 Da, mass difference < 50% of the smaller compound, identical M+H adduct, and one RDKit GetSubstructMatch modification site
- Each retained pair has at least one shifted peak (GNPS greedy bipartite alignment, 40 ppm) whose assigned substructure belongs to the parent compound
- Downstream mean Average-distance Evaluation Scores on the filtered set reproduce MF-N 0.603, MF-R 0.625, and MF-O 0.652 (Fig. 3-A)

## Limitations

- Eligibility restricts the benchmark to pairs with identical M+H adducts and exactly one modification site, so multi-site modifications and non-protonated adducts are excluded
- The 2000 Da precursor cutoff and 50% mass-difference cap limit the chemical diversity of benchmark pairs
- Pairs with no shifted peak explainable by a substructure of the parent compound are discarded (~38% of candidates), biasing evaluation toward spectra with interpretable shifted fragments

## Evidence

- [readme] Tool scope: [evidence span withheld — anchor status: not_found]
