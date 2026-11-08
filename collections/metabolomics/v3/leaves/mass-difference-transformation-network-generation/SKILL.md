---
name: mass-difference-transformation-network-generation
description: Use when FT-ICR MS peak/formula tables are already formula-assigned and
  you need per-sample transformation edge lists and molecular transformation networks
  to probe metabolite connectivity.
when_to_use_negative:
- The peak table has not yet been through formula assignment or MetaboDirect's filtering
  (m/z bounds, 13C isotopes, 0.5 ppm formula error, minimum sample count) — run data
  pre-processing first.
- The goal is identifying metabolites by spectral matching or isomer separation rather
  than inferring transformation relationships; DI-FT-ICR MS cannot separate chemical
  isomers and suffers signal suppression/enhancement that confounds downstream analysis.
- Only a single low-complexity sample with few peaks is available — pairwise difference
  matching against transformation keys is more informative on complex multi-sample
  metabolomic profiles like the bacterium-phage model system.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: MetaboDirect
  role: Command-line pipeline that automates the transformation network generation
    step as part of its 6-step workflow; emits the edge CSVs, counts, and network_stats.R
    script.
  repo: https://github.com/Coayala/MetaboDirect
- name: pandas
  role: Loads the peak/formula table and transf_key.csv and computes pairwise m/z
    differences and 1 ppm matching per sample.
  repo: https://github.com/Coayala/MetaboDirect
- name: NumPy
  role: Array operations backing the pairwise mass-difference calculations alongside
    pandas.
  repo: https://github.com/Coayala/MetaboDirect
- name: NetworkX
  role: Python graph library listed among MetaboDirect's dependencies supporting network
    construction on mass-difference edges.
  repo: https://github.com/Coayala/MetaboDirect
- name: Cytoscape
  role: Imports per-sample edge lists, maps node fill color to molecular class via
    discrete visual style, applies force-directed layout; requires the FileTransfer
    app.
- name: py4cytoscape
  role: Python module used by MetaboDirect to drive Cytoscape for network construction
    and styling.
  repo: https://github.com/Coayala/MetaboDirect
- name: MFAssignR
  role: Upstream tool for automated molecular formula assignment from FT-ICR MS mass
    spectra that produces the peak/formula input consumed by this skill.
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:33f43f867038a8c7576fd80fd0f7923f0a77e8dc3410767a50b5c66e998c1327
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - Cytoscape
  - MFAssignR
  - MetaboDirect
  - NetworkX
  - NumPy
  - Python
  - pandas
  - py4cytoscape
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# mass-difference-transformation-network-generation

## Summary

Generates ab initio biochemical transformation networks for direct-injection FT-ICR MS metabolomic data by matching pairwise peak m/z differences to a predefined transformation key file (max 1 ppm error), then building and visualizing networks whose edges are putative biotic/abiotic transformations.

## When to use

Use when FT-ICR MS peak/formula tables are already formula-assigned and you need per-sample transformation edge lists and molecular transformation networks to probe metabolite connectivity.

## When NOT to use

- The peak table has not yet been through formula assignment or MetaboDirect's filtering (m/z bounds, 13C isotopes, 0.5 ppm formula error, minimum sample count) — run data pre-processing first.
- The goal is identifying metabolites by spectral matching or isomer separation rather than inferring transformation relationships; DI-FT-ICR MS cannot separate chemical isomers and suffers signal suppression/enhancement that confounds downstream analysis.
- Only a single low-complexity sample with few peaks is available — pairwise difference matching against transformation keys is more informative on complex multi-sample metabolomic profiles like the bacterium-phage model system.

## Inputs

- Peak abundance table with assigned molecular formulas from prior FT-ICR MS formula assignment (e.g. MFAssignR or Formularity output)
- Biochemical transformation key file (transf_key.csv) with mass differences and biotic/abiotic classifications from the peat bog study
- Per-sample peak presence/abundance metadata (sample IDs, molecular class assignments)

## Outputs

- Per-sample .csv transformation edge files (nodes = detected peaks, edges = putative mass-based transformations with mass difference, transformation type, and biotic/abiotic label)
- transformation_counts.csv with per-sample counts of matched transformations
- Network statistics .csv tables from network_stats.R
- Bar plots of network metrics as image files
- Cytoscape network files (nodes colored by molecular class, force-directed layout)

## How to apply

After MetaboDirect's pre-processing has filtered peaks by m/z, isotopic 13C presence, formula-assignment error (0.5 ppm) and a user-defined minimum sample count, load the assigned formula/peak table plus the biochemical transformation key file (transf_key.csv) with pandas in the MetaboDirect Python environment. Compute all pairwise m/z differences between peaks within each sample, match each difference to the reference biochemical transformation keys retaining only those with a maximum error of 1 ppm, and classify each retained transformation as biotic or abiotic per the key's classification scheme. Export matched edges as per-sample .csv edge files (nodes = peaks detected in samples, edges = putative chemical transformations) with per-sample counts in transformation_counts.csv. Build networks in Cytoscape from these edge lists, coloring node fill by molecular class via a discrete visual style mapping, and apply a force-directed layout; run the MetaboDirect-generated network_stats.R script to compute per-network and per-transformation-type statistics. Expert review of biological plausibility is the final validation gate, since the 1 ppm match is computational rather than empirical confirmation of a transformation.

## Related tools

- **MetaboDirect** (Command-line pipeline that automates the transformation network generation step as part of its 6-step workflow; emits the edge CSVs, counts, and network_stats.R script.) — https://github.com/Coayala/MetaboDirect
- **pandas** (Loads the peak/formula table and transf_key.csv and computes pairwise m/z differences and 1 ppm matching per sample.) — https://github.com/Coayala/MetaboDirect
- **NumPy** (Array operations backing the pairwise mass-difference calculations alongside pandas.) — https://github.com/Coayala/MetaboDirect
- **NetworkX** (Python graph library listed among MetaboDirect's dependencies supporting network construction on mass-difference edges.) — https://github.com/Coayala/MetaboDirect
- **Cytoscape** (Imports per-sample edge lists, maps node fill color to molecular class via discrete visual style, applies force-directed layout; requires the FileTransfer app.)
- **py4cytoscape** (Python module used by MetaboDirect to drive Cytoscape for network construction and styling.) — https://github.com/Coayala/MetaboDirect
- **MFAssignR** (Upstream tool for automated molecular formula assignment from FT-ICR MS mass spectra that produces the peak/formula input consumed by this skill.)

## Examples

```
metabodirect --input_path my_assigned_formulas.csv --output_path results/ --mode 1
```

## Evaluation signals

- Every per-sample edge .csv exists and is valid CSV carrying the expected transformation columns (mass difference, transformation type, biotic/abiotic label).
- Row counts in transformation_counts.csv equal the number of matched edges in the corresponding per-sample edge file for that sample.
- Every retained edge's mass difference matches a transf_key.csv entry within the 1 ppm maximum error threshold.
- Node counts in the Cytoscape network correspond to peaks detected in the sample after MetaboDirect filtering; nodes carry a molecular class used for fill-color mapping.
- network_stats.R output CSVs contain per-network and per-transformation-type statistics and bar plots render without error.

## Limitations

- The 1 ppm mass-difference match is putative: retained edges are candidate transformations, and biological plausibility requires expert review rather than being guaranteed by the match itself.
- Direct injection FT-ICR MS cannot separate chemical isomers and suffers signal suppression or enhancement, which can confound downstream analysis of the resulting networks.
- Computed clock time grows with sample count and average number of metabolites with assigned molecular formulas; very large datasets may need long runtimes.
- MetaboDirect requires Python ≥3.5, R ≥4, and Cytoscape ≥3.8 with the FileTransfer app and a long list of R packages (tidyverse, vegan, KEGGREST, pmartR, SYNCSA, ggvenn, ggrepel, etc.), so environment setup is non-trivial.
- Transformation classification is limited to the predefined biotic/abiotic keys from the peat bog study; transformations outside that key set are not captured.

## Evidence

- [intro] Transformation networks generated based on mass differences between peaks and predefined biochemical transformation keys: "mass difference network-based approach) are generated in this step, where nodes represent peaks detected in the different samples and edges represent the putative chemical transformations"
- [intro] 1 ppm maximum error against transformation keys: "Mass differences with a maximum error of 1 ppm against the reference biochemical transformation keys are kept as a putative mass-based transformation"
- [readme] Cytoscape networks colored by molecular class with network_stats.R: "MetaboDirect requires Python (3.5 and above), R (4 and above) and Cytoscape (3.8 and above) with the following libraries/modules"
- [intro] Upstream filtering defines which peaks enter the network: [evidence span withheld — anchor status: not_found]
- [results] Isotope filtering removes ~200 masses before network construction: "~200 masses were filtered out from the data set because they were assigned molecular formulas containing an isotopic carbon"
- [abstract] Ab initio network generation is a unique capability of MetaboDirect: "MetaboDirect is also uniquely able to automatically generate biochemical transformation networks (ab initio) based on mass differences (mass difference network-based approach) that provide an"
- [intro] Signal suppression/isomer limitation confounds downstream analysis: "some drawbacks are its inability to separate chemical isomers, lack of fine resolving power, and most importantly signal suppression or enhancement that can confound downstream data analysis"
- [results] Networks visualized in Cytoscape with molecular-class coloring and stats via network_stats.R: [evidence span withheld — anchor status: not_found]
