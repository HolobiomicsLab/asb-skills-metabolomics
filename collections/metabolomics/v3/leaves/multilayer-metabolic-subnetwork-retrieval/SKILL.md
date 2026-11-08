---
name: multilayer-metabolic-subnetwork-retrieval
description: Use when you have completed KGMN network reconstruction (reconstructNetwork1/2/3)
  on MetDNA2 output and need to extract, inspect, or merge the three-layer subnetwork
  for a specific peak pair or centric metabolite (e.g. M182T541–M262T526).
when_to_use_negative:
- Global networks have not yet been reconstructed — reconstructNetwork1/2/3 (with
  reformatTable1 for the annotation table) must run first
- The analysis targets non-model organisms or environmental/exposomics samples where
  organism-specific reaction databases are not yet incorporated into KGMN
- The goal is to distinguish structurally similar isomers of unknown metabolites —
  KGMN cannot effectively discriminate such isomers (e.g. isocitrate glucuronidation
  product isomers)
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: MetDNA2Vis
  role: R package providing retrieveSubNetwork1/2/3 and mergeSubnetwork to extract
    and consolidate the three-layer subnetworks from KGMN output
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Companion ZhuMetLab R package installed alongside MetDNA2Vis and loaded together
    with it for the subnetwork retrieval workflow
  repo: https://github.com/ZhuMetLab/SpectraTools
- name: MetDNA2_Web
  role: Web server implementation of KGMN; its repository hosts the detailed visualization
    tutorial referenced by MetDNA2Vis
  repo: https://github.com/ZhuMetLab/MetDNA2_Web
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:d1291e4e4ee03f758a42f561250ba6bb9ff50f0ce81566387c27b23b05ad8ed9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - MetDNA2Vis
  - MetDNA2_Web
  - SpectraTools
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# multilayer-metabolic-subnetwork-retrieval

## Summary

Extract and consolidate layer-specific subnetworks from the KGMN global networks (metabolic reaction network, peak-to-peak correlation network, MS/MS similarity network) around specific peaks/metabolites using the MetDNA2Vis R package, then merge them into a single resolved subnetwork with conflicting base peaks optimized away.

## When to use

Use when you have completed KGMN network reconstruction (reconstructNetwork1/2/3) on MetDNA2 output and need to extract, inspect, or merge the three-layer subnetwork for a specific peak pair or centric metabolite (e.g. M182T541–M262T526).

## When NOT to use

- Global networks have not yet been reconstructed — reconstructNetwork1/2/3 (with reformatTable1 for the annotation table) must run first
- The analysis targets non-model organisms or environmental/exposomics samples where organism-specific reaction databases are not yet incorporated into KGMN
- The goal is to distinguish structurally similar isomers of unknown metabolites — KGMN cannot effectively discriminate such isomers (e.g. isocitrate glucuronidation product isomers)

## Inputs

- MetDNA2 KGMN project output directory (e.g. Demo_MetDNA2_NIST_urine_pos/06_visualization) containing reconstructed global networks
- Centric metabolite IDs (KEGG IDs like 'C00082' or extended IDs like 'KeggExd000923') for layer-1 retrieval
- Peak identifiers (e.g. 'M182T541', 'M262T526', 'M285T555') and adduct annotations (e.g. '[M+H]+')
- Reformatted annotation table from reformatTable1()

## Outputs

- Layer-1 metabolic-reaction-network subnetwork files in the shared folder_output
- Layer-2 peak-to-peak connection subnetwork files in the shared folder_output
- Layer-3 MS2-similarity subnetwork files in the shared folder_output
- One consolidated merged subnetwork with no remaining conflicting base-peak duplicates

## How to apply

Set the working directory to the MetDNA2 project's 06_visualization folder (e.g. Demo_MetDNA2_NIST_urine_pos/06_visualization) and load MetDNA2Vis and SpectraTools. Run retrieveSubNetwork1 with centric_met (e.g. c('C00082', 'KeggExd000923')) and is_unknown_annotation = TRUE to extract layer-1 metabolic-reaction-network subnetworks while checking empirical rules per subnetwork. Run retrieveSubNetwork2 with from_peak/end_peak (e.g. 'M182T541'/'M262T526') for the layer-2 peak-to-peak connection subnetwork, then retrieveSubNetwork3 with base_peaks and base_adducts (e.g. c('[M+H]+', '[M+H]+')) for the layer-3 MS2-similarity subnetwork. Keep folder_output identical across all three layers (e.g. 'M182T541_M262T526') so the layers resolve together. Finally run mergeSubnetwork with from_peak, end_peak, and the same folder_output to consolidate: when conflicting base peaks arise, retain the larger subnetwork, and merge redundant ion-form peaks of the same metabolite so overlapping annotations collapse into one self-consistent subnetwork.

## Related tools

- **MetDNA2Vis** (R package providing retrieveSubNetwork1/2/3 and mergeSubnetwork to extract and consolidate the three-layer subnetworks from KGMN output) — https://github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Companion ZhuMetLab R package installed alongside MetDNA2Vis and loaded together with it for the subnetwork retrieval workflow) — https://github.com/ZhuMetLab/SpectraTools
- **MetDNA2_Web** (Web server implementation of KGMN; its repository hosts the detailed visualization tutorial referenced by MetDNA2Vis) — https://github.com/ZhuMetLab/MetDNA2_Web

## Examples

```
library(MetDNA2Vis); library(SpectraTools); retrieveSubNetwork1(centric_met = c('C00082','KeggExd000923'), is_unknown_annotation = TRUE, folder_output = 'M182T541_M262T526'); mergeSubnetwork(from_peak = 'M182T541', end_peak = 'M262T526', folder_output = 'M182T541_M262T526')
```

## Evaluation signals

- All three retrieveSubNetwork* calls wrote their files into the same folder_output (e.g. 'M182T541_M262T526'), as the folder_output must stay identical across layers
- After mergeSubnetwork, the merged subnetwork exists in the shared output folder
- All three layers have resolved into one consolidated subnetwork with no remaining conflicting base-peak duplicates
- Conflicting base peaks were resolved by retaining the larger subnetwork, and redundant ion-form peaks of the same metabolite were merged
- checkFiles4Vis(dir_path) and other visualization helpers (convertSpectraData) report the expected files for rendering

## Limitations

- Conflicts between overlapping subnetworks are resolved by maximizing subnetwork self-consistency, so a conflicting annotation may be removed (e.g. M153T555 annotated as 5-ureido-4-imidazole was removed so all peaks merged under xanthosine M285T555)
- KGMN removes some spectrally similar but hard-to-interpret connections, losing interpretive information present in data-driven molecular networks
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites
- Caution is needed when applying KGMN to non-model organisms and environmental/exposomics samples since organism-specific databases are not yet incorporated

## Evidence

- [readme] README example code: "retrieveSubNetwork1(centric_met = c('C00082', 'KeggExd000923'), 
  is_unknown_annotation = TRUE, 
  folder_output = c('M182T541_M262T526'))"
- [readme] README merge step: "# merge subnetwork
mergeSubnetwork(from_peak = 'M182T541', 
  end_peak = 'M262T526', 
  folder_output = 'M182T541_M262T526')"
- [abstract] Three-layer KGMN composition: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [results] Conflict resolution by self-consistency: [evidence span withheld — anchor status: not_found]
- [readme] Folder consistency note: "# Note: the folder_output should keep same among different layer subnetworks"
