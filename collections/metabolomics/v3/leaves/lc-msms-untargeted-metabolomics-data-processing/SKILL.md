---
name: lc-msms-untargeted-metabolomics-data-processing
description: Use when an agent must reproduce or extend KGMN annotation of known and
  unknown metabolites from untargeted LC-MS/MS metabolomics output (e.g., NIST human
  urine positive/negative mode) and quantify how far annotation expands beyond seed
  metabolites.
when_to_use_negative:
- Input is raw LC-MS vendor data not yet processed through KGMN/MetDNA2 — this skill
  visualizes and reproduces existing KGMN annotation output, not raw peak detection
- Samples from non-model organisms or environmental/exposomics profiles, where organism-specific
  reaction databases are not incorporated and the article advises caution
- Scenarios requiring discrimination of structurally similar isomers of unknown metabolites
  (e.g., isocitrate glucuronidation product isomers), which KGMN cannot effectively
  resolve
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MetDNA2Vis
  role: R package used to reconstruct the three KGMN network layers, retrieve and
    merge subnetworks, and visualize/reproduce known and unknown metabolite annotations
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Companion ZhuMetLab R package installed alongside MetDNA2Vis to support spectral
    handling in the annotation reproduction workflow
  repo: ZhuMetLab/SpectraTools
- name: MetDNA2_Web
  role: Web server (http://metdna.zhulab.cn/) and tutorials hosting the KGMN analysis
    whose visualization outputs this skill reproduces
  repo: ZhuMetLab/MetDNA2_Web
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:f48c33bccb06b560a52b100cde61cc20b0b47507334b5691b48ce104b76263cf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetDNA2Vis
  - MetDNA2_Web
  - SpectraTools
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lc-msms-untargeted-metabolomics-data-processing

## Summary

Reproduce KGMN (knowledge-guided multi-layer network) global metabolite annotation from knowns to unknowns on LC-MS/MS untargeted metabolomics data, using MetDNA2Vis to rebuild the three-layer network (KMRN, knowledge-guided MS2 similarity network, global peak correlation network) and tabulate known/unknown peak counts per ionization mode.

## When to use

Use when an agent must reproduce or extend KGMN annotation of known and unknown metabolites from untargeted LC-MS/MS metabolomics output (e.g., NIST human urine positive/negative mode) and quantify how far annotation expands beyond seed metabolites.

## When NOT to use

- Input is raw LC-MS vendor data not yet processed through KGMN/MetDNA2 — this skill visualizes and reproduces existing KGMN annotation output, not raw peak detection
- Samples from non-model organisms or environmental/exposomics profiles, where organism-specific reaction databases are not incorporated and the article advises caution
- Scenarios requiring discrimination of structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers), which KGMN cannot effectively resolve

## Inputs

- Exported KGMN/MetDNA2 results directory, e.g. Demo_MetDNA2_NIST_urine_pos/06_visualization/
- MS1 peak table and MS2 spectra from untargeted LC-MS/MS metabolomics data (positive and/or negative ionization modes)
- Seed metabolite annotations matched against standard spectral libraries (HMDB/PubChem-backed libraries)
- Annotation table produced by reformatTable1()

## Outputs

- Merged global annotation network (from mergeSubnetwork) spanning all three KGMN layers
- Per-layer subnetwork exports in folder_output (e.g. 'M182T541_M262T526')
- kgmn_nist_urine_stats.csv with seed, peak, and known/unknown counts per ionization mode
- Annotated peaks split into knowns and unknowns (e.g. 927 peaks = 634 knowns + 293 unknowns in positive mode)

## How to apply

Set the working directory to the exported KGMN results folder 'Demo_MetDNA2_NIST_urine_pos/06_visualization/' and install SpectraTools and MetDNA2Vis from GitHub (ZhuMetLab). Build layer 1 by running reconstructNetwork1(is_unknown_annotation = TRUE) to expand seed metabolites (matched by MS1 m/z and MS2 spectrum against standard spectral libraries) over the knowledge-based metabolic reaction network (KMRN); reformat the annotation table with reformatTable1() and build layer 2 with reconstructNetwork2(annotation_table = annotation_table), the knowledge-guided MS2 similarity network; then build layer 3, the global peak correlation network, with reconstructNetwork3(). Export per-layer subnetworks into a shared folder_output (e.g., 'M182T541_M262T526') using retrieveSubNetwork1 (with centric metabolites like 'C00082' and 'KeggExd000923'), retrieveSubNetwork2 (from_peak 'M182T541', end_peak 'M262T526'), and retrieveSubNetwork3 (base_peaks with adducts '[M+H]+'), then merge them with mergeSubnetwork(). Finally tabulate seed, peak, and known/unknown counts per mode (positive: 173 seeds, 927 peaks = 634 knowns + 293 unknowns; negative: 161 seeds, 1283 peaks = 652 knowns + 631 unknowns) and validate exact equality against the reported NIST urine statistics.

## Related tools

- **MetDNA2Vis** (R package used to reconstruct the three KGMN network layers, retrieve and merge subnetworks, and visualize/reproduce known and unknown metabolite annotations) — github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Companion ZhuMetLab R package installed alongside MetDNA2Vis to support spectral handling in the annotation reproduction workflow) — ZhuMetLab/SpectraTools
- **MetDNA2_Web** (Web server (http://metdna.zhulab.cn/) and tutorials hosting the KGMN analysis whose visualization outputs this skill reproduces) — ZhuMetLab/MetDNA2_Web

## Examples

```
library(MetDNA2Vis); library(SpectraTools); setwd('Demo_MetDNA2_NIST_urine_pos/06_visualization/'); reconstructNetwork1(is_unknown_annotation = TRUE); annotation_table <- reformatTable1(); reconstructNetwork2(annotation_table = annotation_table); reconstructNetwork3(); retrieveSubNetwork2(from_peak = 'M182T541', end_peak = 'M262T526', folder_output = 'M182T541_M262T526'); mergeSubnetwork(from_peak = 'M182T541', end_peak = 'M262T526', folder_output = 'M182T541_M262T526')
```

## Evaluation signals

- Positive-mode NIST urine counts equal exactly 173 seed metabolites and 927 total annotated peaks split as 634 knowns + 293 unknowns
- Negative-mode NIST urine counts equal exactly 161 seeds and 1283 peaks split as 652 knowns + 631 unknowns
- The three reconstructNetwork1/2/3 calls complete and retrieveSubNetwork1/2/3 write subnetworks into the same folder_output ('M182T541_M262T526') before mergeSubnetwork succeeds
- Total peaks annotated (927 in positive mode) expand well beyond the 173 seed metabolites, consistent with KGMN's expected ~100–300 putative unknowns per data set
- Tabulated statistics written to kgmn_nist_urine_stats.csv match reported NIST urine values with exact equality for seeds and known/unknown splits

## Limitations

- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers)
- Unknowns are absent from spectral databases and lack chemical standards, so validating unknown annotations remains a grand challenge; KGMN also removes spectrally similar but hard-to-interpret connections from data-driven molecular networks

## Evidence

- [abstract] The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network.: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [results] KGMN annotated 927 peaks (634 knowns, 293 unknowns) in NIST urine positive mode from 173 seeds: [evidence span withheld — anchor status: not_found]
- [readme] README example invocation reconstructing the three KGMN network layers: [evidence span withheld — anchor status: not_found]
- [readme] README note that folder_output must be kept consistent across layer subnetworks: "# Note: the folder_output should keep same among different layer subnetworks"
- [discussion] Article discusses caution for non-model organisms and exposomics samples: "one needs to be cautious when applying it to some cases like non-model organisms and environmental and exposomics-related profiles"
