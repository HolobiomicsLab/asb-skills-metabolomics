---
name: known-unknown-peak-annotation-classification
description: Use when a completed KGMN/MetDNA2 annotation run must be summarized as
  seed/peak/known/unknown counts per ionization mode, or when reproducing published
  peak-annotation statistics from exported visualization folders.
when_to_use_negative:
- Input is raw LC-MS data or a feature table that has not been processed through KGMN/MetDNA2
  annotation (no exported 06_visualization folder exists)
- The goal is de novo structure elucidation or in silico MS/MS prediction (MetFrag,
  CFM-ID, MS-FINDER, SIRIUS) rather than counting known/unknown annotations from a
  KGMN run
- Applying to non-model organisms or environmental/exposomics samples, where the article
  cautions organism-specific reaction databases are not yet incorporated
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MetDNA2Vis
  role: R package that reconstructs the three KGMN network layers, retrieves and merges
    subnetworks, and supplies the annotation tables used to count knowns vs unknowns
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Companion ZhuMetLab R package installed alongside MetDNA2Vis to support spectra
    handling during network reconstruction
  repo: https://github.com/ZhuMetLab/SpectraTools
- name: MetDNA2_Web
  role: Web server and tutorial source for KGMN annotation runs whose outputs this
    skill classifies
  repo: https://github.com/ZhuMetLab/MetDNA2_Web
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:2687dcfbf11b694479c388864d6c4bbca43ac50c1f34a7f59b00c5de2d3b4366
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - MetDNA2Vis
  - MetDNA2_Web
  - SpectraTools
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# known-unknown-peak-annotation-classification

## Summary

Tabulates how KGMN's three-layer network (knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, global peak correlation network) splits annotated MS1 peaks into knowns versus unknowns for a given ionization mode, e.g. 927 peaks (634 knowns + 293 unknowns) from 173 seed metabolites in NIST human urine positive mode.

## When to use

Use when a completed KGMN/MetDNA2 annotation run must be summarized as seed/peak/known/unknown counts per ionization mode, or when reproducing published peak-annotation statistics from exported visualization folders.

## When NOT to use

- Input is raw LC-MS data or a feature table that has not been processed through KGMN/MetDNA2 annotation (no exported 06_visualization folder exists)
- The goal is de novo structure elucidation or in silico MS/MS prediction (MetFrag, CFM-ID, MS-FINDER, SIRIUS) rather than counting known/unknown annotations from a KGMN run
- Applying to non-model organisms or environmental/exposomics samples, where the article cautions organism-specific reaction databases are not yet incorporated

## Inputs

- Exported KGMN/MetDNA2 result folder (e.g. Demo_MetDNA2_NIST_urine_pos/06_visualization/) containing per-layer network and annotation tables
- Annotation table produced by reformatTable1() from the layer-1 network
- Known seed-peak identifiers (e.g. centric_met 'C00082', 'KeggExd000923'; peaks 'M182T541', 'M262T526' with adducts '[M+H]+')

## Outputs

- Merged annotation subnetwork from mergeSubnetwork() (per-layer subnetworks in folder_output)
- kgmn_nist_urine_stats.csv with seed metabolite, MS1 peak, and known/unknown peak counts per ionization mode
- Validated peak-classification statistics (positive: 927 = 634 knowns + 293 unknowns; negative: 1283 = 652 knowns + 631 unknowns)

## How to apply

Set the working directory to the exported demo data folder (e.g. 'Demo_MetDNA2_NIST_urine_pos/06_visualization/') and rebuild the multilayer network with MetDNA2Vis: reconstructNetwork1(is_unknown_annotation = TRUE) expands the 173 seed metabolites over the knowledge-based metabolic reaction network; reformatTable1() then reconstructNetwork2(annotation_table = ...) builds the knowledge-guided MS2 similarity network; reconstructNetwork3() adds the global peak correlation network. Retrieve and merge per-layer subnetworks (retrieveSubNetwork1/2/3 with a shared folder_output such as 'M182T541_M262T526', then mergeSubnetwork(from_peak = 'M182T541', end_peak = 'M262T526')). Finally tabulate seed, MS1 peak, and known/unknown peak counts per ionization mode and validate against expected statistics — positive mode: 173 seeds, 927 peaks = 634 knowns + 293 unknowns; negative mode: 161 seeds, 1283 peaks = 652 knowns + 631 unknowns — requiring exact equality.

## Related tools

- **MetDNA2Vis** (R package that reconstructs the three KGMN network layers, retrieves and merges subnetworks, and supplies the annotation tables used to count knowns vs unknowns) — https://github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Companion ZhuMetLab R package installed alongside MetDNA2Vis to support spectra handling during network reconstruction) — https://github.com/ZhuMetLab/SpectraTools
- **MetDNA2_Web** (Web server and tutorial source for KGMN annotation runs whose outputs this skill classifies) — https://github.com/ZhuMetLab/MetDNA2_Web

## Examples

```
setwd('Demo_MetDNA2_NIST_urine_pos/06_visualization/'); reconstructNetwork1(is_unknown_annotation = TRUE); annotation_table <- reformatTable1(); reconstructNetwork2(annotation_table = annotation_table); reconstructNetwork3()
```

## Evaluation signals

- Positive-mode counts must match reported values exactly: 173 seed metabolites, 927 annotated peaks, 634 knowns, 293 unknowns
- Negative-mode counts must match: 161 seeds, 1283 peaks, 652 knowns, 631 unknowns (knowns + unknowns = total peaks in both modes)
- Peak totals must expand well beyond the seed metabolite count (927 peaks from 173 seeds), confirming network propagation occurred
- folder_output identifier must be identical across retrieveSubNetwork1/2/3 and mergeSubnetwork calls (e.g. 'M182T541_M262T526') for the merged network to form correctly
- kgmn_nist_urine_stats.csv is written and its rows reconcile with the reported NIST urine statistics

## Limitations

- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g. isocitrate glucuronidation product isomers)
- Unknowns are not in spectral databases and lack chemical standards, so validating unknown annotations remains a grand challenge; only five recurrent unknowns were validated with synthetic standards

## Evidence

- [results] findings: [evidence span withheld — anchor status: not_found]
- [readme] readme: "reconstructNetwork1(is_unknown_annotation = TRUE)"
- [abstract] abstract: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [discussion] discussion: "Since unknown metabolites are not included in spectral databases and have no chemical standards, validation of unknown annotations remains a grand challenge"
