---
name: linear-mixed-effects-modeling-of-protein-abundances
description: Use when protein-level abundance matrices from twin/pair-related or longitudinally
  repeated DIA (SWATH) measurements must be decomposed into heritable, environmental,
  and longitudinal variance components.
when_to_use_negative:
- Input is already a variance-components table or the per-protein abundance matrix
  has not been produced from OpenSWATH targeted analysis of the DIA maps
- The data are not twin/pair-related or longitudinally repeated measurements, so heritable
  vs environmental vs longitudinal components cannot be distinguished
- Absolute protein abundance quantification is the goal rather than decomposition
  of variance in already-quantified abundances
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: OpenMS 2.0
  role: Provides the upstream processing chain (file conversion, centroiding, database
    searching, feature detection, RT alignment, cross-run feature linking) that produces
    the per-protein abundance matrix modeled by the mixed-effects fit
  repo: https://github.com/OpenMS/OpenMS
- name: OpenMS
  role: OpenSWATH targeted extraction of peptide transitions from plasma maps with
    decoy-based scoring feeds the abundance matrix entering the variance-component
    model
  repo: https://github.com/OpenMS/OpenMS
- name: Python
  role: Fits the per-protein linear mixed-effects model that partitions abundance
    variance into heritable, environmental, and longitudinal components
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:9da2bf355c9b12ba20f48a983ce1ed90a1bad9e3afeec623db91d8623cf139e2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - OpenMS
  - OpenMS 2.0
  - Python
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Linear mixed-effects modeling of protein abundances

## Summary

Fits a per-protein linear mixed-effects model to SWATH-MS plasma protein abundances, partitioning observed variance into heritable (shared between co-twins), common/individual environmental, and longitudinal (time) components, as demonstrated in the Liu et al. twin-cohort analysis with OpenMS/OpenSWATH-derived quantities.

## When to use

Use when protein-level abundance matrices from twin/pair-related or longitudinally repeated DIA (SWATH) measurements must be decomposed into heritable, environmental, and longitudinal variance components.

## When NOT to use

- Input is already a variance-components table or the per-protein abundance matrix has not been produced from OpenSWATH targeted analysis of the DIA maps
- The data are not twin/pair-related or longitudinally repeated measurements, so heritable vs environmental vs longitudinal components cannot be distinguished
- Absolute protein abundance quantification is the goal rather than decomposition of variance in already-quantified abundances

## Inputs

- Twin-cohort blood plasma SWATH-MS (DIA) mass spectrometric maps
- Assay/transition library for OpenSWATH targeted extraction
- Per-protein abundance matrix from OpenSWATH/OpenMS targeted analysis
- Python statistical environment for linear mixed-effects fitting

## Outputs

- Per-protein variance fractions attributable to heritable, environmental (common and individual), and longitudinal components
- variance_components.csv table with heritable/environmental/longitudinal variance partitioning

## How to apply

Aggregate OpenSWATH-extracted peptide quantities from twin-cohort blood plasma DIA maps into a per-protein abundance matrix via protein inference and statistical error control (using the OpenMS 2.0 chain: file conversion, centroiding, database searching, feature detection, RT alignment, cross-run feature linking). For each protein, fit a linear mixed-effects model in Python whose random effects encode co-twin sharing (heritable component), shared/individual environment, and longitudinal time effects. Extract the variance attributed to each component and normalize per protein so components sum to ~1, producing a variance_components.csv table. Expert review resolves residuals where the three components do not sum to approximately 1 or diverge from the paper's reported partitioning.

## Related tools

- **OpenMS 2.0** (Provides the upstream processing chain (file conversion, centroiding, database searching, feature detection, RT alignment, cross-run feature linking) that produces the per-protein abundance matrix modeled by the mixed-effects fit) — https://github.com/OpenMS/OpenMS
- **OpenMS** (OpenSWATH targeted extraction of peptide transitions from plasma maps with decoy-based scoring feeds the abundance matrix entering the variance-component model) — https://github.com/OpenMS/OpenMS
- **Python** (Fits the per-protein linear mixed-effects model that partitions abundance variance into heritable, environmental, and longitudinal components)

## Examples

```
python variance_decomposition.py --input abundance_matrix.csv --output variance_components.csv --components heritable,environmental,longitudinal
```

## Evaluation signals

- Heritable + environmental (common and individual) + longitudinal components sum to approximately 1 per protein
- Decomposition is consistent with the paper's reported per-protein partitioning; residual discrepancies resolved by expert review
- Over 300 plasma proteins quantified, matching the OpenSWATH scale reported by Liu et al.
- variance_components.csv contains one row per protein with the three variance fractions

## Limitations

- Requires twin/pair-related and longitudinal sampling structure; single-time-point, unrelated-cohort designs cannot support the three-component decomposition
- Discrepancies where the three variance components do not sum to ~1 require expert review rather than an automated fix
- Fidelity depends on the quality of OpenSWATH targeted extraction, RT alignment, and cross-run feature linking of the plasma DIA maps

## Evidence

- [other] decompose the observed variance for each protein into heritable, environmental (common and individual) and longitudinal components: [evidence span withheld — anchor status: not_found]
- [readme] chain of OpenMS 2.0 steps feeding the abundance matrix: [evidence span withheld — anchor status: external_doc]
- [abstract] OpenMS 2.0 targeted DIA quantification: "We present OpenMS 2.0 (http://www.openms.org), a robust open-source, cross-platform software specifically designed for the flexible and reproducible analysis of high-throughput MS data."
