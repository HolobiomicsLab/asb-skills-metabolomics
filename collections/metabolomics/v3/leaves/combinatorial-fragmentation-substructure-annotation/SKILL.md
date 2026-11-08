---
name: combinatorial-fragmentation-substructure-annotation
description: Use when MS/MS peaks of a known compound must be annotated with candidate
  substructures (e.g. to localize a structural modification site from shifted peaks),
  and you have a 2D structure (SMILES/InChI) plus peak list.
when_to_use_negative:
- The compound has no reliable 2D structure — combinatorial fragmentation requires
  a molecular graph to enumerate substructures.
- You need a full structure-spectrum prediction rather than site localization (e.g.
  CFM-ID-style in-silico spectra for the whole molecule).
edam_operation: http://edamontology.org/operation_3636
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ModiFinder
  role: Wraps MAGMa combinatorial fragmentation (breadth-first, breaks/depth parameter)
    to produce peak→substructure annotations consumed by the likelihood engine.
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MAGMa
  role: Reference combinatorial fragmentation algorithm; defines the breadth-first
    substructure enumeration and m/z assignment used at depth 2.
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_006
  - task_007
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:81ca5d1682edb0e23d55b2793506f716d4051dfddbe5243b3d2495d9f3550230
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - MAGMa
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3636
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# combinatorial-fragmentation-substructure-annotation

## Summary

Assign each MS/MS peak a set of possible substructures by combinatorial fragmentation (MAGMa-style breadth-first traversal of the known compound's molecular graph up to depth 2), then keep only substructures whose theoretical m/z falls within 40 ppm of the empirical peak — the annotation substrate ModiFinder needs before computing modification-site likelihoods.

## When to use

Use when MS/MS peaks of a known compound must be annotated with candidate substructures (e.g. to localize a structural modification site from shifted peaks), and you have a 2D structure (SMILES/InChI) plus peak list.

## When NOT to use

- The compound has no reliable 2D structure — combinatorial fragmentation requires a molecular graph to enumerate substructures.
- You need a full structure-spectrum prediction rather than site localization (e.g. CFM-ID-style in-silico spectra for the whole molecule).

## Inputs

- MS/MS spectrum of a known compound (peak list with m/z and intensity, e.g. from MGF/mzML)
- 2D structure of the known compound (SMILES/InChI) to enumerate candidate substructures
- m/z tolerance (40 ppm) and fragmentation depth (2 default)

## Outputs

- Peak-to-fragment map: for each MS/MS peak, a set of assigned substructures (fragment formulas and subgraph node sets)
- Counts of annotated shifted peaks per pair for downstream likelihood scoring

## How to apply

Start from the initial structure (S-known) and perform breadth-first fragmentation up to a predetermined depth — a maximum depth of 2 in the published workflow. For each peak, assign any substructure whose theoretical m/z (protonated, charge assumed 1) falls within 40 ppm of the empirical m/z; the 40 ppm tolerance matches the spectral alignment used to derive shifted peaks. Before annotation, preprocess the spectrum by removing peaks below 1% of the base-peak intensity and normalizing to Euclidean norm 1. Iterate the depth (2 vs 3 vs 4) as a sensitivity check: deeper fragmentation increases the fraction of annotated shifted peaks (≈19% from depth 2 to 4) but also inflates ambiguity, so the depth choice trades recall of shifted peaks against precision of site likelihood. In oracle mode, filter out substructures assigned to shifted peaks that do not contain the true modification site.

## Related tools

- **ModiFinder** (Wraps MAGMa combinatorial fragmentation (breadth-first, breaks/depth parameter) to produce peak→substructure annotations consumed by the likelihood engine.) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MAGMa** (Reference combinatorial fragmentation algorithm; defines the breadth-first substructure enumeration and m/z assignment used at depth 2.)

## Examples

```
from modi_finder import MAGMaAnnotationEngine; engine = MAGMaAnnotationEngine(breaks=2, max_mz_ppm=40); annotations = engine.annotate(spectrum, smiles)
```

## Evaluation signals

- Reproduced shifted-peak counts match the reported depth-2 baseline (8,033 of 12,909 benchmark pairs retained with ≥1 explainable shifted peak, 62%).
- Depth sweep shows the expected ~19% increase in annotated shifted peaks from depth 2 to 4.
- Every assigned substructure's theoretical m/z is within 40 ppm of its peak (charge = 1).
- Oracle filtering removes no unshifted-peak substructures; shifted-peak substructures lacking the true site vanish in MF-O mode.
- Likelihood post-processing (threshold 0.5, power 4) yields the expected MF-N/MF-R/MF-O Average-distance scores of 0.603/0.625/0.652.

## Limitations

- Depth 2 default caps substructure recall; deeper fragmentation (3–4) raises shifted-peak annotation but increases ambiguity of site likelihoods.
- m/z matching is charge-assumed-1 and 40 ppm wide, so isobaric substructures can be over-assigned (expert peak-to-fragment pruning was needed for Naphthomycin B: 27–40 substructures reduced to 2).
- Substructures without the true modification site cannot be distinguished without oracle or helper refinement — purity depends on the quality of S-known.

## Evidence

- [results] Depth sweep finding (results): [evidence span withheld — anchor status: not_found]
- [results] Expert pruning of over-annotation: [evidence span withheld — anchor status: not_found]
