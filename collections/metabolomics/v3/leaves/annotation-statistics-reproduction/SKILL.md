---
name: annotation-statistics-reproduction
description: Use when a task requires exactly reproducing published KGMN annotation
  counts (seeds, knowns, unknowns) from NIST urine or similar untargeted LC-MS metabolomics
  data using MetDNA2Vis output folders.
when_to_use_negative:
- The input is raw LC-MS data (mzML/CDF) rather than an exported MetDNA2/KGMN result
  folder — run the KGMN annotation workflow first
- The goal is de novo unknown structure elucidation rather than reproducing the published
  known/unknown annotation statistics
- The sample is from a non-model organism or environmental/exposomics study, where
  the paper advises caution because organism-specific reaction databases are not incorporated
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetDNA2Vis
  role: Reconstructs the three KGMN network layers, retrieves and merges subnetworks,
    and provides checkFiles4Vis to validate exported result folders
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Companion ZhuMetLab R package installed alongside MetDNA2Vis for the visualization/reproduction
    workflow
  repo: https://github.com/ZhuMetLab/SpectraTools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:09345db89c89b62c46789b1643c7ddbdc4bd6656e10de38ff585b6a0623afeed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - MetDNA2Vis
  - SpectraTools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# annotation-statistics-reproduction

## Summary

Reproduces the KGMN (knowledge-guided multi-layer network) global metabolite annotation statistics from the Nature Communications 2022 paper by re-running the MetDNA2Vis three-layer network reconstruction on the exported demo result folder (e.g. Demo_MetDNA2_NIST_urine_pos/06_visualization/) and tabulating seed, known, and unknown peak counts per ionization mode.

## When to use

Use when a task requires exactly reproducing published KGMN annotation counts (seeds, knowns, unknowns) from NIST urine or similar untargeted LC-MS metabolomics data using MetDNA2Vis output folders.

## When NOT to use

- The input is raw LC-MS data (mzML/CDF) rather than an exported MetDNA2/KGMN result folder — run the KGMN annotation workflow first
- The goal is de novo unknown structure elucidation rather than reproducing the published known/unknown annotation statistics
- The sample is from a non-model organism or environmental/exposomics study, where the paper advises caution because organism-specific reaction databases are not incorporated

## Inputs

- Exported KGMN demo result folder (e.g. Demo_MetDNA2_NIST_urine_pos/06_visualization/)
- KGMN annotation tables per ionization mode (positive and negative)
- Seed metabolite list (173 seeds in positive mode, 161 in negative mode)

## Outputs

- kgmn_nist_urine_stats.csv with per-mode seed, peak, known, and unknown counts
- Reconstructed layer 1/2/3 global networks (KMRN, MS2 similarity network, peak correlation network)
- Merged annotation subnetwork (e.g. M182T541_M262T526)

## How to apply

Set the R working directory to the exported KGMN demo folder 'Demo_MetDNA2_NIST_urine_pos/06_visualization/'. Build layer 1 with MetDNA2Vis::reconstructNetwork1(is_unknown_annotation = TRUE) to expand the 173 seed metabolites over the knowledge-based metabolic reaction network (KMRN); reformat the annotation table with reformatTable1() and build layer 2, the knowledge-guided MS2 similarity network, with reconstructNetwork2(annotation_table = annotation_table); build layer 3, the global peak correlation network, with reconstructNetwork3(). Optionally retrieve per-layer subnetworks for a peak pair (e.g. from_peak = 'M182T541', end_peak = 'M262T526') using retrieveSubNetwork1/2/3 and mergeSubnetwork, keeping folder_output identical ('M182T541_M262T526') across layers. Then tabulate seed metabolite, MS1 peak, and known/unknown peak counts per ionization mode (positive: 173 seeds, 927 peaks = 634 knowns + 293 unknowns; negative: 161 seeds, 1283 peaks = 652 knowns + 631 unknowns) and write kgmn_nist_urine_stats.csv, validating exact equality against the reported statistics.

## Related tools

- **MetDNA2Vis** (Reconstructs the three KGMN network layers, retrieves and merges subnetworks, and provides checkFiles4Vis to validate exported result folders) — https://github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Companion ZhuMetLab R package installed alongside MetDNA2Vis for the visualization/reproduction workflow) — https://github.com/ZhuMetLab/SpectraTools

## Examples

```
reconstructNetwork1(is_unknown_annotation = TRUE); annotation_table <- reformatTable1(); reconstructNetwork2(annotation_table = annotation_table); reconstructNetwork3()
```

## Evaluation signals

- Positive-mode counts must match exactly: 173 seed metabolites and 927 total annotated peaks = 634 knowns + 293 unknowns
- Negative-mode counts must match exactly: 161 seeds and 1283 total peaks = 652 knowns + 631 unknowns
- folder_output must be identical across retrieveSubNetwork1/2/3 and mergeSubnetwork calls (e.g. 'M182T541_M262T526')
- checkFiles4Vis(dir_path) passes on the exported demo folder before network reconstruction
- kgmn_nist_urine_stats.csv is written with the expected seed/peak/known/unknown columns and values

## Limitations

- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g. isocitrate glucuronidation product isomers)
- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy
- Validating unknown annotations remains a grand challenge because unknowns are absent from spectral databases and lack chemical standards

## Evidence

- [abstract] The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network.: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [intro] a knowledge-based metabolic reaction network (KMRN), a knowledge-guided MS2 similarity network, and a global peak correlation network: "The KGMN approach integrated three layers of networks, including a knowledge-based metabolic reaction network (KMRN), a knowledge-guided MS2 similarity network, and a global peak correlation network"
- [methods] devtools::install_github("ZhuMetLab/MetDNA2Vis"): "devtools::install_github("ZhuMetLab/SpectraTools")
devtools::install_github("ZhuMetLab/MetDNA2Vis")"
- [readme] Note: the folder_output should keep same among different layer subnetworks: "Note: the folder_output should keep same among different layer subnetworks"
