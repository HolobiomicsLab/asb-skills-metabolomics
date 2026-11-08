---
name: biotic-abiotic-transformation-classification
description: Use when building MetaboDirect mass-difference transformation networks
  from FT-ICR MS peak/formula tables and each candidate edge must be labeled biotic
  or abiotic per the transformation key scheme.
when_to_use_negative:
- The peak table has not yet been through MetaboDirect pre-processing filters (m/z
  filtering, 13C isotopic peak removal, 0.5 ppm formula-assignment error cutoff),
  since unfiltered masses will inflate spurious matches
- The goal is de novo metabolite identification rather than mass-difference-based
  transformation network edge annotation
- Data are not direct injection high-resolution MS (e.g., FT-ICR MS) peak/formula
  data with assigned molecular formulas
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: MetaboDirect
  role: Pipeline whose transformation network analysis step performs the pairwise
    mass-difference matching and biotic/abiotic classification; source code and user's
    guide at its GitHub repository
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Loads the peak/formula table and transformation key file and computes per-sample
    pairwise m/z differences
- name: NumPy
  role: Supports pairwise mass-difference computation between peaks within each sample
- name: NetworkX
  role: Supports construction of the mass-difference transformation networks whose
    edges are the classified transformations
- name: FT-ICR MS
  role: High-resolution mass spectrometry technique producing the peak data the mass
    differences are computed from
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:48654784d976776b9789d16a9cc063b6e6fbd2fca634ebbb2b0d0bdfe568827f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - FT-ICR MS
  - MetaboDirect
  - NetworkX
  - NumPy
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# biotic-abiotic-transformation-classification

## Summary

Classifies putative mass-based biochemical transformations in MetaboDirect transformation networks as biotic or abiotic, using mass differences computed between all peaks in each sample and matched (≤1 ppm error) against a predefined biochemical transformation key file.

## When to use

Use when building MetaboDirect mass-difference transformation networks from FT-ICR MS peak/formula tables and each candidate edge must be labeled biotic or abiotic per the transformation key scheme.

## When NOT to use

- The peak table has not yet been through MetaboDirect pre-processing filters (m/z filtering, 13C isotopic peak removal, 0.5 ppm formula-assignment error cutoff), since unfiltered masses will inflate spurious matches
- The goal is de novo metabolite identification rather than mass-difference-based transformation network edge annotation
- Data are not direct injection high-resolution MS (e.g., FT-ICR MS) peak/formula data with assigned molecular formulas

## Inputs

- Assigned molecular formula / peak abundance table from FT-ICR MS data (post formula assignment, post peak filtering)
- Biochemical transformation key file (transf_key.csv) with mass differences and biotic/abiotic classification labels

## Outputs

- Per-sample .csv edge files for the transformation network (edges = putative chemical transformations, nodes = peaks detected in the samples)
- transformation_counts.csv tabulating per-sample transformation counts
- Biotic/abiotic label per retained transformation edge

## How to apply

Load the assigned molecular formula/peak table and the biochemical transformation key file (transf_key.csv) with pandas inside the MetaboDirect Python environment. Compute all pairwise m/z differences between peaks detected within each sample using pandas/NumPy, then match each mass difference against the reference biochemical transformation keys, keeping only differences with a maximum error of 1 ppm as putative mass-based transformations. Classify each retained transformation as biotic or abiotic according to the transformation key classification scheme from the peat bog study. Export matched edges (nodes = peaks detected in the samples, edges = putative chemical transformations) as per-sample .csv files, and tabulate per-sample transformation counts into transformation_counts.csv. Note this step runs downstream of peak filtering (m/z values, 13C isotopic peaks, 0.5 ppm formula-assignment error, user-defined minimum sample count), so apply it to the filtered peak table.

## Related tools

- **MetaboDirect** (Pipeline whose transformation network analysis step performs the pairwise mass-difference matching and biotic/abiotic classification; source code and user's guide at its GitHub repository) — https://github.com/Coayala/MetaboDirect
- **pandas** (Loads the peak/formula table and transformation key file and computes per-sample pairwise m/z differences)
- **NumPy** (Supports pairwise mass-difference computation between peaks within each sample)
- **NetworkX** (Supports construction of the mass-difference transformation networks whose edges are the classified transformations)
- **FT-ICR MS** (High-resolution mass spectrometry technique producing the peak data the mass differences are computed from)

## Examples

```
metabodirect -h
```

## Evaluation signals

- Each per-sample edge .csv file exists and is valid CSV with the expected transformation columns (e.g., mass difference, transformation type, biotic/abiotic label)
- Per-sample counts in transformation_counts.csv equal the number of matched edges for that sample
- Every retained edge's mass difference matches a transformation key with error ≤ 1 ppm; no edge exceeds this tolerance
- Every edge label is one of the biotic/abiotic categories defined in the transformation key classification scheme
- Network biological plausibility is assessed by expert_review on the classified transformation networks

## Limitations

- Mass-difference-based matching is putative: edges represent putative chemical transformations, not confirmed reactions, so biological plausibility requires expert review
- Direct injection MS cannot separate chemical isomers and suffers signal suppression or enhancement, which can confound downstream analysis of the networks
- The 1 ppm error tolerance and the biotic/abiotic scheme derive from the transformation key file (peat bog study classification), so keys outside that reference set cannot be classified
- Isotopic carbon (13C) assignments can create false mass differences if not filtered beforehand (~200 masses were removed from the bacterium-phage data set for this reason)

## Evidence

- [intro] 1 ppm threshold: "Mass differences with a maximum error of 1 ppm against the reference biochemical transformation keys are kept as a putative mass-based transformation"
- [intro] network nodes/edges definition: "nodes represent peaks detected in the different samples and edges represent the putative chemical transformations"
- [intro] filter step context: [evidence span withheld — anchor status: not_found]
- [abstract] unique capability claim: "MetaboDirect is also uniquely able to automatically generate biochemical transformation networks (ab initio) based on mass differences (mass difference network-based approach)"
- [readme] install/run context from README: "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above) with the following libraries/modules"
