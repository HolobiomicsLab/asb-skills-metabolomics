---
name: benchmark-dataset-construction-and-qc
description: Use when building a reference set of structurally related MS/MS pairs
  for benchmarking fragment-peak alignment or modification-site localization.
when_to_use_negative:
- Input spectra are not library MS/MS spectra of structural analogs with a known compound/unknown
  compound pairing (e.g., de novo spectra from a biological sample).
- Only unmodified single-compound spectra need to be analyzed; benchmarking pair-based
  analog alignment is unnecessary.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ModiFinder
  role: Provides the pair-alignment, MAGMa combinatorial fragmentation, and shifted-peak
    explainability logic used to filter benchmark pairs
  repo: github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Combinatorial fragmentation engine invoked (breadth-first, max depth 2) to
    assign candidate substructures to peaks within 40 ppm
- name: MS/MS library search
  role: Source of the GNPS library spectra and compound pairs that constitute the
    benchmark database
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:34e100fbe504eb4317bac50d0eb4da6727382c31a1617f4a2c902b36ef6574e3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MAGMa
  - MS/MS library search
  - ModiFinder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# benchmark-dataset-construction-and-qc

## Summary

Construct and quality-control a paired MS/MS benchmark of structurally related analog compounds from GNPS spectral libraries, with explicit eligibility checks and a shifted-peak explainability filter, to evaluate modification-site localization.

## When to use

Use when building a reference set of structurally related MS/MS pairs for benchmarking fragment-peak alignment or modification-site localization.

## When NOT to use

- Input spectra are not library MS/MS spectra of structural analogs with a known compound/unknown compound pairing (e.g., de novo spectra from a biological sample).
- Only unmodified single-compound spectra need to be analyzed; benchmarking pair-based analog alignment is unnecessary.

## Inputs

- GNPS spectral library entries (mzML/MGF-style MS/MS spectra and compound metadata from GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB)
- Precursor mass, adduct (M+H), and structural metadata per library compound

## Outputs

- Raw analog pairs database (~12,909 benchmark pairs passing eligibility checks)
- filtered_pairs.csv containing the 8,033 retained pairs (62% of the 12,909 benchmark pairs)
- Per-pair records of aligned shifted and unshifted matched peaks

## How to apply

Build a benchmark database of analog MS/MS pairs from the four GNPS spectral libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB). Enforce the eligibility checks: precursor mass < 2000 Da, precursor mass difference < 50%, same adduct (M+H), and exactly one modification site. Then align MS2-known and MS2-unknown spectra via the GNPS bipartite-graph greedy matching at 40 ppm tolerance, assign substructures with MAGMa-style combinatorial fragmentation (breadth-first, maximum depth 2) within 40 ppm, and keep a pair only if at least one shifted peak is explainable by a substructure of the known compound. This filtered benchmark yields 8,033 retained pairs, representing 62% of the 12,909 constructed benchmark pairs.

## Related tools

- **ModiFinder** (Provides the pair-alignment, MAGMa combinatorial fragmentation, and shifted-peak explainability logic used to filter benchmark pairs) — github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Combinatorial fragmentation engine invoked (breadth-first, max depth 2) to assign candidate substructures to peaks within 40 ppm)
- **MS/MS library search** (Source of the GNPS library spectra and compound pairs that constitute the benchmark database)

## Examples

```
python run_modifinder.py --pairs 12909 --output filtered_pairs.csv
```

## Evaluation signals

- Retained pair count should be 8,033 out of 12,909 benchmark pairs (62%).
- All retained pairs must satisfy precursor mass < 2000 Da, precursor mass difference < 50%, same M+H adduct, and exactly one modification site.
- Every retained pair must have at least one shifted peak with a substructure assigned within 40 ppm.
- Peak preprocessing should leave peaks with intensity ≥1% of the base peak, normalized to a Euclidean norm of 1.

## Limitations

- Benchmark construction is tied to the specific four GNPS libraries used (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB) and may not generalize to other libraries.
- Eligibility checks exclude precursor masses ≥ 2000 Da, precursor mass differences ≥ 50%, and pairs with more than one modification site, narrowing the pair universe.
- The 40 ppm tolerance and max depth of 2 for combinatorial fragmentation are the parameters used here; different values may change the retained fraction.
- Web tool performance demonstration used a separate dataset (TUEBINGEN-NATURAL-PRODUCT-COLLECTION), so the benchmark is not the only evaluation set.

## Evidence
