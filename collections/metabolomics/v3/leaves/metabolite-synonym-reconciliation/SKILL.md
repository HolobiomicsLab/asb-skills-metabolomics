---
name: metabolite-synonym-reconciliation
description: Use when two metabolite annotation engines (e.g. SpaMTP and METASPACE/pySM)
  annotate overlapping m/z peaks from the same spatial metabolomics dataset and synonym
  naming differences must be reconciled before computing per-database match percentages.
when_to_use_negative:
- The goal is a single-engine annotation run (SpaMTP AnnotateSM alone) with no cross-tool
  benchmark against METASPACE annotations
- The data are high-resolution FT-ICR or orbitrap acquisitions where METASPACE/pySM
  applies directly, so no platform-agnostic reconciliation is needed
- Fully automated FDR-filtered annotation is required and no manual synonym adjudication
  step is feasible (this skill's reconciliation step is explicitly manual)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SpaMTP
  role: Annotates m/z peaks platform-agnostically via AnnotateSM (±3 ppm, HMDB/LipidMaps)
    and provides LoadSM/AnnotateBigData used to build the annotation side of the benchmark
  repo: github.com/GenomicsMachineLearning/SpaMTP
- name: METASPACE
  role: Source of the comparison annotations (FDR ≤ 20%, HMDB and LipidMaps) whose
    metabolite names are reconciled against SpaMTP's at shared m/z masses
- name: HMDB
  role: Reference database searched by both engines; synonym reconciliation determines
    the 87% match rate over shared masses
- name: LIPIDMAPS
  role: Reference database searched by both engines; synonym reconciliation determines
    the 93% match rate over shared masses
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:2f5d1ea8acaae3ee3143eb7a98b62cb8247ceb8cad3c071e470c54c825c79f9d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Cardinal
  - HMDB
  - LIPIDMAPS
  - METASPACE
  - SpaMTP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# metabolite-synonym-reconciliation

## Summary

Benchmark metabolite annotations from SpaMTP's AnnotateSM pipeline against METASPACE/pySM annotations on shared m/z peaks, manually reconciling synonyms of the same metabolite so per-database match rates (e.g. HMDB 87%, LipidMaps 93%) can be computed.

## When to use

Use when two metabolite annotation engines (e.g. SpaMTP and METASPACE/pySM) annotate overlapping m/z peaks from the same spatial metabolomics dataset and synonym naming differences must be reconciled before computing per-database match percentages.

## When NOT to use

- The goal is a single-engine annotation run (SpaMTP AnnotateSM alone) with no cross-tool benchmark against METASPACE annotations
- The data are high-resolution FT-ICR or orbitrap acquisitions where METASPACE/pySM applies directly, so no platform-agnostic reconciliation is needed
- Fully automated FDR-filtered annotation is required and no manual synonym adjudication step is feasible (this skill's reconciliation step is explicitly manual)

## Inputs

- MALDI-TOF spatial metabolomics dataset in imzML format (e.g. human lung dataset with 401,325 detected m/z values)
- SpaMTP annotation output from AnnotateSM over HMDB and LipidMaps (normalised internal reference format, ±3 ppm tolerance)
- Previously published METASPACE annotations for the same dataset at maximum FDR 20% (HMDB and LipidMaps)

## Outputs

- Benchmark table (lung_annotation_benchmark.csv) with per-database annotation counts for SpaMTP and METASPACE and the fraction of matching metabolite names among shared m/z masses
- Reconciled list of overlapping m/z peaks with matched/reconciled metabolite names across engines

## How to apply

Load the spatial metabolomics dataset with SpaMTP's LoadSM (wrapping Cardinal's readImzML) and confirm the full m/z list (401,325 detected values in the human lung MALDI-TOF case). Run AnnotateSM on every valid m/z with a ±3 ppm exact-mass tolerance, searching HMDB and LipidMaps converted to the normalised internal reference format in the correct polarity/adduct space. Retrieve METASPACE annotations for the same dataset at a maximum FDR of 20% per database. Restrict comparison to overlapping m/z masses between the two engines, and during reconciliation treat synonyms of the same metabolite manually (e.g. HMDB-style vs LipidMaps-style names for the same compound) rather than relying on string equality. Finally compute, per database, SpaMTP-annotated counts (~80,235 HMDB; ~39,215 LipidMaps), METASPACE-annotated counts (247 HMDB; 322 LipidMaps), and the fraction of matching metabolite names among shared masses, assembling the counts and match percentages into a single benchmark table (lung_annotation_benchmark.csv). The rationale is that SpaMTP annotates m/z peaks agnostic of the data-generating platform, whereas METASPACE/pySM is optimised for high-resolution instruments (FT-ICR/orbitrap) and applies FDR-based filtering, so direct comparison requires normalising naming at shared masses.

## Related tools

- **SpaMTP** (Annotates m/z peaks platform-agnostically via AnnotateSM (±3 ppm, HMDB/LipidMaps) and provides LoadSM/AnnotateBigData used to build the annotation side of the benchmark) — github.com/GenomicsMachineLearning/SpaMTP
- **METASPACE** (Source of the comparison annotations (FDR ≤ 20%, HMDB and LipidMaps) whose metabolite names are reconciled against SpaMTP's at shared m/z masses)
- **HMDB** (Reference database searched by both engines; synonym reconciliation determines the 87% match rate over shared masses)
- **LIPIDMAPS** (Reference database searched by both engines; synonym reconciliation determines the 93% match rate over shared masses)

## Examples

```
devtools::install_github("GenomicsMachineLearning/SpaMTP")
```

## Evaluation signals

- SpaMTP-annotated m/z counts land near the published values (~80,235 for HMDB and ~39,215 for LipidMaps)
- METASPACE-annotated counts match the published values (247 for HMDB and 322 for LipidMaps at FDR ≤ 20%)
- Match percentage among shared masses is near ~90% (HMDB 87%, LipidMaps 93%) after manual synonym reconciliation
- Only overlapping m/z masses between SpaMTP and METASPACE are compared, with synonyms of the same metabolite counted as matches
- Final benchmark table contains per-database counts and match percentages assembled into lung_annotation_benchmark.csv

## Limitations

- Synonym reconciliation is manual: 'treating synonyms of the same metabolite manually during reconciliation' — no automated synonym mapping is specified, so results are adjudicator-dependent
- METASPACE/pySM is optimised for high-resolution FT-ICR/orbitrap data and does not apply to most TOF-acquired data, so the comparison side of the benchmark depends on previously published annotations rather than a rerun
- No explicit SpaMTP version number is given in the source material, complicating exact reproduction of annotation counts
- Match percentages are computed against METASPACE's FDR-filtered (≤20%) annotations, so raw counts are not directly comparable without the shared-mass restriction

## Evidence

- [other] Match overlapping m/z masses between SpaMTP and METASPACE by comparing corresponding metabolite annotations, treating synonyms of the same metabolite manually during reconciliation: [evidence span withheld — anchor status: not_found]
