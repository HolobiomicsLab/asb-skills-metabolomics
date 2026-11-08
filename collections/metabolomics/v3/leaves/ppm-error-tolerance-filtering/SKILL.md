---
name: ppm-error-tolerance-filtering
description: Use when building mass-difference transformation networks from direct-injection
  FT-ICR MS peak tables and you must decide which pairwise m/z differences match predefined
  biochemical transformation keys within a ppm error threshold.
when_to_use_negative:
- Input is raw unprocessed FT-ICR MS spectra — molecular formula assignment must already
  have been performed before this skill applies
- The task is formula assignment itself (e.g., using MFAssignR) rather than matching
  already-assigned mass differences to transformation keys
- The analysis requires chromatographic separation of isomers — direct injection FT-ICR
  MS cannot separate chemical isomers and suffers signal suppression/enhancement
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetaboDirect
  role: Pipeline whose transformation-network step applies the 1 ppm mass-difference
    matching against biochemical transformation keys
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Loads peak/transformation-key tables and computes pairwise m/z differences
    per sample
- name: NumPy
  role: Numerical computation of mass differences and ppm errors
- name: NetworkX
  role: Constructs the mass-difference networks from filtered edges
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:ae1108bb003b46619198ec5bfc513a8044a1489df2c2d3dd0b895fbd5d68dba8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - MetaboDirect
  - NetworkX
  - NumPy
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ppm-error-tolerance-filtering

## Summary

Applies parts-per-million (ppm) error tolerances to decide which FT-ICR MS mass differences count as putative biochemical transformations (max 1 ppm) and which peaks/formula assignments survive pre-processing (0.5 ppm formula error) in MetaboDirect's transformation network workflow.

## When to use

Use when building mass-difference transformation networks from direct-injection FT-ICR MS peak tables and you must decide which pairwise m/z differences match predefined biochemical transformation keys within a ppm error threshold.

## When NOT to use

- Input is raw unprocessed FT-ICR MS spectra — molecular formula assignment must already have been performed before this skill applies
- The task is formula assignment itself (e.g., using MFAssignR) rather than matching already-assigned mass differences to transformation keys
- The analysis requires chromatographic separation of isomers — direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression/enhancement

## Inputs

- Peak abundance table with assigned molecular formulas from FT-ICR MS (CSV)
- Biochemical transformation key file (transf_key.csv) with reference transformation masses and biotic/abiotic classifications
- User-defined m/z filter bounds, isotopic-peak removal flag, and minimum-sample-count threshold

## Outputs

- Per-sample edge .csv files (nodes = peaks detected in samples, edges = putative chemical transformations with mass difference, transformation type, biotic/abiotic label)
- transformation_counts.csv tabulating per-sample transformation counts
- Transformed network structures for downstream NetworkX/Cytoscape analysis

## How to apply

Load the formula-assigned peak table and the biochemical transformation key file (transf_key.csv) with pandas in the MetaboDirect Python environment, after peaks have been filtered by m/z range, 13C isotopic presence, formula-assignment error (0.5 ppm), and minimum sample count. Compute all pairwise m/z differences between peaks within each sample using pandas/NumPy, then match each mass difference against the reference biochemical transformation keys, retaining only those with a maximum error of 1 ppm as putative mass-based transformations. Classify each retained transformation as biotic or abiotic according to the transformation-key classification scheme from the peat bog study. Rationale: the 1 ppm tolerance balances the ultra-high mass accuracy of FT-ICR MS against false matches to transformation keys, while the stricter 0.5 ppm filter at the formula-assignment stage removes unreliable formula annotations before differences are computed.

## Related tools

- **MetaboDirect** (Pipeline whose transformation-network step applies the 1 ppm mass-difference matching against biochemical transformation keys) — https://github.com/Coayala/MetaboDirect
- **pandas** (Loads peak/transformation-key tables and computes pairwise m/z differences per sample)
- **NumPy** (Numerical computation of mass differences and ppm errors)
- **NetworkX** (Constructs the mass-difference networks from filtered edges)

## Examples

```
metabodirect -h
```

## Evaluation signals

- Every retained mass difference in the per-sample edge files has an error ≤ 1 ppm against its matched transformation key
- Each per-sample edge .csv exists, parses as valid CSV, and contains the expected transformation columns (mass difference, transformation type, biotic/abiotic label)
- Per-sample counts in transformation_counts.csv equal the number of matched edges in the corresponding per-sample edge file
- Peaks removed upstream: formula assignments with error > 0.5 ppm, 13C isotopic masses, and peaks below the user-defined minimum sample count are absent from node sets
- Network biological plausibility is assessed by expert_review after export

## Limitations

- Direct-injection MS cannot separate chemical isomers and signal suppression/enhancement can confound downstream analysis
- Isotopic-carbon-containing masses must be filtered out (~200 masses removed from the bacterium-phage data set) or they corrupt transformation matching
- If the input data set was already filtered upstream (as with the S. fallax data set), re-filtering may be unnecessary or counterproductive

## Evidence

- [intro] Mass differences with a maximum error of 1 ppm against the reference biochemical transformation keys are kept as a putative mass-based transformation: "Mass differences with a maximum error of 1 ppm against the reference biochemical transformation keys are kept as a putative mass-based transformation"
- [intro] Peaks filtered by m/z, isotopes, formula error: [evidence span withheld — anchor status: not_found]
- [intro] Nodes are peaks, edges are putative transformations: "mass difference network-based approach) are generated in this step, where nodes represent peaks detected in the different samples and edges represent the putative chemical transformations"
- [readme] README: FT-ICR MS pipeline scope: "A comprehensive command-line based pipeline for the analysis of direct injection FT-ICR mass spectrometry data."
