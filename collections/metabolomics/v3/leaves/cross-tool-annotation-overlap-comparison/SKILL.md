---
name: cross-tool-annotation-overlap-comparison
description: 'Decision support for choosing an annotation tool for MALDI/SM data:
  run when a user asks whether SpaMTP''s annotation covers METASPACE/pySM results
  on TOF data, or needs per-database overlap statistics between two annotation pipelines.'
when_to_use_negative:
- Input is a high-resolution instrument (orbitrap/FT-ICR) dataset where METASPACE/pySM
  is already the optimised default and SpaMTP's agnostic annotation is not the object
  of comparison
- Dataset or database lacks a defined m/z peak list, or the comparison tool's annotations
  are unavailable for the same sample, making head-to-head overlap undefined
- Only one tool's annotation output exists and no cross-tool reconciliation is requested
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3360
tools:
- name: SpaMTP
  role: Performs the automated m/z annotation (AnnotateSM) and provides the platform-agnostic
    annotation layer being benchmarked
  repo: https://github.com/GenomicsMachineLearning/SpaMTP
- name: METASPACE
  role: Provides the independent reference annotations (at a stated FDR) that SpaMTP's
    annotations are matched against
- name: Cardinal
  role: Underlying reader for imzML SM data loaded through SpaMTP's LoadSM wrapper
- name: pySM
  role: Python package for automated FDR-based metabolite annotation; the tool whose
    METASPACE-coupled annotations serve as the comparison baseline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.10.31.621429
    title: 'SpaMTP: Integrative Statistical Analysis and Visualisation of Spatial
      Metabolomics and Transcriptomics data'
  source_content_hash: sha256:a4daa378bfa52cfddd108769351515e784f6a3cce2805e6250cdaa0cac831ddf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GenomicsMachineLearning/SpaMTP
  tools:
  - Cardinal
  - HMDB
  - LIPIDMAPS
  - METASPACE
  - SpaMTP
  - pySM
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cross-tool-annotation-overlap-comparison

## Summary

Benchmark protocol for comparing automated m/z metabolite annotations from SpaMTP against an independent tool's annotations (e.g. METASPACE/pySM) on the same spatial metabolomics dataset, quantifying per-database annotation counts and metabolite-name match fractions among shared masses.

## When to use

Decision support for choosing an annotation tool for MALDI/SM data: run when a user asks whether SpaMTP's annotation covers METASPACE/pySM results on TOF data, or needs per-database overlap statistics between two annotation pipelines.

## When NOT to use

- Input is a high-resolution instrument (orbitrap/FT-ICR) dataset where METASPACE/pySM is already the optimised default and SpaMTP's agnostic annotation is not the object of comparison
- Dataset or database lacks a defined m/z peak list, or the comparison tool's annotations are unavailable for the same sample, making head-to-head overlap undefined
- Only one tool's annotation output exists and no cross-tool reconciliation is requested

## Inputs

- Spatial metabolomics dataset in imzML format (or SpaMTP-readable equivalent) with a known m/z peak list
- Reference metabolite database in SpaMTP's normalised internal format (e.g. HMDB, LipidMaps)
- Independent tool's annotations for the same dataset (e.g. METASPACE results at a stated FDR)

## Outputs

- Benchmark table (e.g. CSV) with per-database counts of m/z values annotated by each tool and the fraction of matching metabolite names among shared masses
- Synonym-reconciled metabolite name matches between the two annotation sets

## How to apply

Load the target SM dataset with SpaMTP (e.g. via Cardinal's imzML reader wrapped by LoadSM) and enumerate all detected m/z values, then run SpaMTP's AnnotateSM annotation pipeline with a defined exact-mass error tolerance (the paper's lung example uses ±3 ppm) against a specified reference database (HMDB, LipidMaps) in the correct polarity/adduct space. Independently retrieve the comparison tool's annotations (e.g. METASPACE at a stated FDR threshold, e.g. 20%) for the same dataset from its published source or via pySM. Match overlapping m/z masses between the two annotation sets by comparing corresponding metabolite annotations; treat synonyms of the same metabolite manually during reconciliation. Compute, per database, the count of m/z values annotated by each tool and the fraction of matching metabolite names among shared masses, then assemble into a single benchmark table (the paper's lung benchmark reports ~80,235 HMDB and ~39,215 LipidMaps SpaMTP annotations vs METASPACE's 247 and 322, with ~90% name concordance: 87% HMDB, 93% LipidMaps). Rationale: SpaMTP is platform-agnostic and applies to TOF data, whereas METASPACE/pySM is optimised for high-resolution instruments such as orbitrap or FT-ICR, so a head-to-head count plus name-match concordance quantifies both breadth and correctness.

## Related tools

- **SpaMTP** (Performs the automated m/z annotation (AnnotateSM) and provides the platform-agnostic annotation layer being benchmarked) — https://github.com/GenomicsMachineLearning/SpaMTP
- **METASPACE** (Provides the independent reference annotations (at a stated FDR) that SpaMTP's annotations are matched against)
- **Cardinal** (Underlying reader for imzML SM data loaded through SpaMTP's LoadSM wrapper)
- **pySM** (Python package for automated FDR-based metabolite annotation; the tool whose METASPACE-coupled annotations serve as the comparison baseline)

## Examples

```
lung_bench <- AnnotateSM(sm_data, database = list(HMDB, LipidMaps), ppm_error = 3); compare with METASPACE annotations at FDR 20% and write lung_annotation_benchmark.csv
```

## Evaluation signals

- Per-database counts of annotated m/z values fall in the expected ranges reported by the paper (e.g. ~80,235 HMDB, ~39,215 LipidMaps for SpaMTP vs 247/322 for METASPACE on the human lung MALDI-TOF dataset)
- Fraction of matching metabolite names among shared m/z masses is high (~90% expected: 87% HMDB, 93% LipidMaps)
- All m/z values from the input dataset are accounted for in the benchmark table (annotated vs unannotated per tool per database)
- Synonym reconciliation is documented — metabolite names that differ but refer to the same compound are treated as matches, not mismatches

## Limitations

- METASPACE and pySM are optimised for high-resolution instruments such as FT-ICR or orbitrap and do not yet apply to most data obtained with TOF analysers, limiting direct comparability on TOF data
- Manually reconciling metabolite synonyms is required when matching masses between tools, and this step is subjective and not automated
- Comparison only covers exact-mass/name overlap; it does not capture differences in scoring, confidence, or downstream usability between tools

## Evidence

- [other] Research question framing: does SpaMTP annotation work directly on data regardless of platform?: [evidence span withheld — anchor status: not_found]
- [other] Workflow definition for annotation benchmark: [evidence span withheld — anchor status: not_found]
- [readme] SpaMTP tool role in annotation: [evidence span withheld — anchor status: external_doc]
