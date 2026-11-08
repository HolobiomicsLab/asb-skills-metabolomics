---
name: ft-icr-ms-pairwise-mass-difference-matching
description: Decision support for building ab initio biochemical transformation networks
  from FT-ICR MS peak tables that already have assigned molecular formulas; triggered
  when you need putative biotic/abiotic edges (e.g., CH2, N2 gains/losses) per sample.
when_to_use_negative:
- Input is raw FT-ICR MS spectra without molecular formula assignment — MetaboDirect
  requires formula-assigned peak abundance data produced by upstream tools (e.g.,
  MFAssignR or Formularity).
- The input data is LC-MS/MS or DI-MS where isomers are separated; this skill targets
  direct injection FT-ICR MS peak tables and does not resolve chemical isomers.
- You need transformation edges with strict stoichiometric validation beyond mass-difference
  tolerance; this approach matches mass differences at ≤1 ppm and does not independently
  confirm biochemical plausibility beyond the reference keys.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MetaboDirect
  role: Pipeline that computes pairwise m/z differences, matches transformation keys,
    and exports per-sample transformation networks
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Loads peak tables and computes pairwise m/z differences per sample
- name: NumPy
  role: Numerical computation of pairwise differences and ppm error calculation
- name: NetworkX
  role: Builds and visualizes the transformation networks from matched edges
- name: Cytoscape
  role: Optional interactive inspection of the per-sample transformation networks
    (requires ≥3.8 and the FileTransfer app)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:070bdee6449fc6f421957c3dcda019d9d09a6fc0e2b6a8a89f3f1430de763b24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Cytoscape
  - MetaboDirect
  - NetworkX
  - NumPy
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# ft-icr-ms-pairwise-mass-difference-matching

## Summary

Computes all pairwise m/z differences among FT-ICR MS-detected peaks in each sample, matches them against a reference biochemical transformation key file at ≤1 ppm error, and exports per-sample edge files plus transformation counts for building molecular transformation networks.

## When to use

Decision support for building ab initio biochemical transformation networks from FT-ICR MS peak tables that already have assigned molecular formulas; triggered when you need putative biotic/abiotic edges (e.g., CH2, N2 gains/losses) per sample.

## When NOT to use

- Input is raw FT-ICR MS spectra without molecular formula assignment — MetaboDirect requires formula-assigned peak abundance data produced by upstream tools (e.g., MFAssignR or Formularity).
- The input data is LC-MS/MS or DI-MS where isomers are separated; this skill targets direct injection FT-ICR MS peak tables and does not resolve chemical isomers.
- You need transformation edges with strict stoichiometric validation beyond mass-difference tolerance; this approach matches mass differences at ≤1 ppm and does not independently confirm biochemical plausibility beyond the reference keys.

## Inputs

- Peak abundance table with assigned molecular formulas (post-processing FT-ICR MS output, per sample)
- Biochemical transformation key file (transf_key.csv) with reference mass differences and biotic/abiotic labels
- Sample metadata / per-sample peak lists

## Outputs

- Per-sample .csv edge files (nodes = peaks, edges = putative chemical transformations with mass difference, transformation identity, and biotic/abiotic label)
- transformation_counts.csv aggregating the number of matched transformations per sample
- Molecular transformation networks consumable by Cytoscape or NetworkX

## How to apply

Load the post-formula-assignment peak table (peak abundance plus assigned molecular formula, after MetaboDirect pre-processing filters such as 0.5 ppm formula-assignment error and removal of 13C isotopic peaks) and the biochemical transformation key file (transf_key.csv) into the MetaboDirect Python environment using pandas/NumPy. For each sample, compute all pairwise m/z differences between detected peaks and match each difference against the reference biochemical transformation keys, keeping only differences within a maximum error of 1 ppm as putative mass-based transformations. Classify each retained transformation as biotic or abiotic according to the transformation key classification scheme (derived from the peat bog study), with nodes representing peaks and edges representing the putative chemical transformations. Export the matched edges as per-sample .csv edge files and aggregate per-sample transformation counts into transformation_counts.csv; the resulting networks can be inspected in Cytoscape (≥3.8 with the FileTransfer app) or rendered via NetworkX within the pipeline.

## Related tools

- **MetaboDirect** (Pipeline that computes pairwise m/z differences, matches transformation keys, and exports per-sample transformation networks) — https://github.com/Coayala/MetaboDirect
- **pandas** (Loads peak tables and computes pairwise m/z differences per sample)
- **NumPy** (Numerical computation of pairwise differences and ppm error calculation)
- **NetworkX** (Builds and visualizes the transformation networks from matched edges)
- **Cytoscape** (Optional interactive inspection of the per-sample transformation networks (requires ≥3.8 and the FileTransfer app))

## Examples

```
metabodirect -d my_project_dir -i peak_abundance.csv -m metadata.csv --transf_key transf_key.csv
```

## Evaluation signals

- Each per-sample edge .csv exists, parses as valid CSV, and contains expected transformation columns (mass difference, transformation identity, biotic/abiotic label).
- Row count in transformation_counts.csv for a sample equals the number of matched edges exported for that sample.
- Every retained mass difference has |error| ≤ 1 ppm against the reference transformation key mass.
- No edge links peaks filtered out during pre-processing (e.g., 13C isotopic peaks or >0.5 ppm formula-assignment error).
- Networks open in Cytoscape / NetworkX without orphaned nodes or malformed edge attributes.

## Limitations

- Maximum error tolerance is fixed at 1 ppm; looser or stricter tolerances are not configurable within this matching step.
- Direct injection FT-ICR MS cannot separate chemical isomers, so distinct isobaric transformations sharing the same mass difference are indistinguishable.
- Signal suppression or enhancement can confound downstream analysis, potentially biasing edge frequency.
- Transformation plausibility depends entirely on the provided transf_key.csv reference keys; keys not present in the file will never be matched.

## Evidence

- [intro] Mass differences with a maximum error of 1 ppm against the reference biochemical transformation keys are kept as a putative mass-based transformation: "Mass differences with a maximum error of 1 ppm against the reference biochemical transformation keys are kept as a putative mass-based transformation"
- [intro] nodes represent peaks detected in the different samples and edges represent the putative chemical transformations: "mass difference network-based approach) are generated in this step, where nodes represent peaks detected in the different samples and edges represent the putative chemical transformations"
- [intro] The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra or any other high-resolution MS technique: "The pipeline accepts peak abundance and assigned molecular formula data produced after an initial processing of raw FT-ICR MS spectra or any other high-resolution MS technique"
- [abstract] MetaboDirect is also uniquely able to automatically generate biochemical transformation networks (ab initio) based on mass differences (mass difference network-based approach) that provide an: "MetaboDirect is also uniquely able to automatically generate biochemical transformation networks (ab initio) based on mass differences (mass difference network-based approach) that provide an"
- [readme] MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above): "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above)"
