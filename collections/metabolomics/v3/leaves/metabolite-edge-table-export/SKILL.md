---
name: metabolite-edge-table-export
description: Use when FT-ICR MS peak tables with assigned molecular formulas are available
  and ab initio biochemical transformation networks based on mass differences must
  be built and exported per sample.
when_to_use_negative:
- 'Input is raw FT-ICR MS spectra: molecular formula assignment must be performed
  before MetaboDirect, so this skill cannot start from unprocessed spectra'
- The peak table has not been filtered by m/z values, 13C isotopic peaks, formula-assignment
  error (0.5 ppm), and minimum sample presence — unfiltered tables will propagate
  spurious isotopic masses (~200 removed in the bacterium-phage dataset) into edges
- The goal is formula-level pathway annotation (KEGG Pathway/Module/Brite via KEGGREST)
  rather than mass-difference transformation networks
tools:
- name: MetaboDirect
  role: Pipeline providing the transformation network analysis step that computes
    mass-difference edges and exports per-sample edge files
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Loading peak/formula tables and transf_key.csv; computing pairwise per-sample
    m/z differences and writing per-sample .csv edge files
- name: NumPy
  role: Vectorized pairwise m/z difference computation between peaks within each sample
- name: NetworkX
  role: Downstream construction of the transformation network whose edges are the
    exported putative chemical transformations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:47fcbfa77cb5eb310ccaa53bbb1f9f0cf628081756b58a6b79739d703467532d
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# metabolite-edge-table-export

## Summary

Reconstructs MetaboDirect's mass-difference transformation phase: compute all pairwise m/z differences between peaks within each FT-ICR MS sample, match them to predefined biochemical transformation keys at a maximum error of 1 ppm, label each edge biotic or abiotic, and export per-sample edge .csv files plus a transformation_counts.csv.

## When to use

Use when FT-ICR MS peak tables with assigned molecular formulas are available and ab initio biochemical transformation networks based on mass differences must be built and exported per sample.

## When NOT to use

- Input is raw FT-ICR MS spectra: molecular formula assignment must be performed before MetaboDirect, so this skill cannot start from unprocessed spectra
- The peak table has not been filtered by m/z values, 13C isotopic peaks, formula-assignment error (0.5 ppm), and minimum sample presence — unfiltered tables will propagate spurious isotopic masses (~200 removed in the bacterium-phage dataset) into edges
- The goal is formula-level pathway annotation (KEGG Pathway/Module/Brite via KEGGREST) rather than mass-difference transformation networks

## Inputs

- Peak abundance table with assigned molecular formulas from FT-ICR MS (direct injection), one table per sample
- Reference biochemical transformation key file (transf_key.csv) with biotic/abiotic classification from the peat bog study

## Outputs

- Per-sample .csv edge files (edges = putative chemical transformations with mass difference, transformation type, biotic/abiotic label; nodes = peaks detected in the samples)
- transformation_counts.csv tabulating per-sample transformation counts

## How to apply

Load the assigned molecular formula/peak table and the biochemical transformation key file (transf_key.csv) with pandas inside the MetaboDirect Python environment. Compute all pairwise m/z differences between peaks detected within each sample using pandas/NumPy, then match each difference against the reference biochemical transformation keys, keeping only those with a maximum error of 1 ppm as putative mass-based transformations. Classify each retained transformation as biotic or abiotic according to the transformation key classification scheme from the peat bog study. Export the matched edges (nodes = peaks detected in the samples, edges = putative chemical transformations) as per-sample .csv files and tabulate per-sample transformation counts into transformation_counts.csv for downstream network construction (e.g., with NetworkX/Cytoscape).

## Related tools

- **MetaboDirect** (Pipeline providing the transformation network analysis step that computes mass-difference edges and exports per-sample edge files) — https://github.com/Coayala/MetaboDirect
- **pandas** (Loading peak/formula tables and transf_key.csv; computing pairwise per-sample m/z differences and writing per-sample .csv edge files)
- **NumPy** (Vectorized pairwise m/z difference computation between peaks within each sample)
- **NetworkX** (Downstream construction of the transformation network whose edges are the exported putative chemical transformations)

## Examples

```
pip install metabodirect; metabodirect -h  # CLI entry point documented in the README; run the main pipeline to trigger transformation network analysis and per-sample edge exports
```

## Evaluation signals

- Each per-sample edge .csv exists and is valid CSV with expected transformation columns (mass difference, transformation type, biotic/abiotic label)
- Per-sample counts in transformation_counts.csv equal the number of matched edges in the corresponding per-sample edge file
- Every retained edge's mass difference matched a transformation key within a maximum error of 1 ppm
- Edge set contains no differences derived from 13C isotopic peaks or peaks failing the 0.5 ppm formula-assignment filter
- Network biological plausibility of the exported transformation edges is assessed by expert_review

## Limitations

- Biotic/abiotic classification depends on the transformation key scheme inherited from the peat bog study, so labels are scheme-specific rather than universal
- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression or enhancement, which can confound downstream analysis of the resulting networks
- Mass-difference edges are putative only: matching a mass difference to a transformation key does not confirm the biochemical reaction occurred in the sample

## Evidence

- [intro] Mass differences with a maximum error of 1 ppm against the reference biochemical transformation keys are kept as a putative mass-based transformation: "Mass differences with a maximum error of 1 ppm against the reference biochemical transformation keys are kept as a putative mass-based transformation"
- [intro] nodes represent peaks detected in the different samples and edges represent the putative chemical transformations: "mass difference network-based approach) are generated in this step, where nodes represent peaks detected in the different samples and edges represent the putative chemical transformations"
- [abstract] MetaboDirect is also uniquely able to automatically generate biochemical transformation networks (ab initio) based on mass differences: "MetaboDirect is also uniquely able to automatically generate biochemical transformation networks (ab initio) based on mass differences (mass difference network-based approach)"
- [intro] detected peaks are filtered by their m/z values (based on the user's input), isotopic presence (13C peaks), error in formula assignment (0.5 ppm): [evidence span withheld — anchor status: not_found]
- [readme] MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above): "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above) with the following libraries/modules"
