---
name: r-package-workflow-execution
description: Use when reproducing or auditing KGMN subnetwork reconstruction from
  MetDNA2 output (e.g., the 06_visualization folder of Demo_MetDNA2_NIST_urine_pos)
  with the MetDNA2Vis/SpectraTools R packages.
when_to_use_negative:
- Input is raw LC-MS data rather than a completed MetDNA2 output directory — KGMN
  network construction must run first
- Data are from non-model organisms or environmental/exposomics profiles, where the
  authors caution against direct application because organism-specific databases are
  not yet incorporated
- The goal is distinguishing structurally similar isomers of unknown metabolites,
  which KGMN cannot resolve effectively (e.g., isocitrate glucuronidation product
  isomers)
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MetDNA2Vis
  role: R package that reconstructs the three KGMN network layers, extracts layer
    subnetworks, and merges them via reconstructNetwork1-3, retrieveSubNetwork1-3,
    and mergeSubnetwork
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: R package installed alongside MetDNA2Vis to support spectral handling in the
    workflow
  repo: https://github.com/ZhuMetLab/SpectraTools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:2fd435643fd701f2dde1f409bb755066ea98d26a3dbdb06a925645bd29c71174
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - MetDNA2Vis
  - SpectraTools
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# r-package-workflow-execution

## Summary

Executes the MetDNA2Vis + SpectraTools R workflow to reconstruct the three KGMN network layers, extract per-subnetwork layer files, and consolidate them via mergeSubnetwork in the Demo_MetDNA2_NIST_urine_pos demo, reproducing the KGMN subnetwork optimization and filtering logic.

## When to use

Use when reproducing or auditing KGMN subnetwork reconstruction from MetDNA2 output (e.g., the 06_visualization folder of Demo_MetDNA2_NIST_urine_pos) with the MetDNA2Vis/SpectraTools R packages.

## When NOT to use

- Input is raw LC-MS data rather than a completed MetDNA2 output directory — KGMN network construction must run first
- Data are from non-model organisms or environmental/exposomics profiles, where the authors caution against direct application because organism-specific databases are not yet incorporated
- The goal is distinguishing structurally similar isomers of unknown metabolites, which KGMN cannot resolve effectively (e.g., isocitrate glucuronidation product isomers)

## Inputs

- MetDNA2 result folder Demo_MetDNA2_NIST_urine_pos/06_visualization (including annotation_table and MS2 data)
- Peak identifiers (e.g., 'M182T541', 'M262T526') and centric metabolite IDs (e.g., 'C00082', 'KeggExd000923')
- Base adduct specifications such as '[M+H]+' for MS2-similarity subnetwork extraction

## Outputs

- Layer-1 metabolic-reaction-network subnetwork files in folder_output
- Layer-2 peak-to-peak connection subnetwork files in the same folder_output
- Layer-3 MS2-similarity subnetwork files in the same folder_output
- One consolidated merged subnetwork in the shared output folder (e.g., 'M182T541_M262T526') with no conflicting base-peak duplicates

## How to apply

Install the ZhuMetLab packages via devtools::install_github('ZhuMetLab/SpectraTools') and devtools::install_github('ZhuMetLab/MetDNA2Vis'), plus CRAN/Bioconductor dependencies (dplyr, tidyr, readr, CHNOSZ, igraph, magrittr, ggplot2, ggraph, tidygraph). Set the working directory to the demo's 06_visualization folder, then build the layers in order: reconstructNetwork1(is_unknown_annotation = TRUE), reformatTable1() followed by reconstructNetwork2(annotation_table = annotation_table), and reconstructNetwork3(). Extract each subnetwork layer with matching folder_output (e.g., 'M182T541_M262T526'): retrieveSubNetwork1(centric_met = c('C00082', 'KeggExd000923'), is_unknown_annotation = TRUE), retrieveSubNetwork2(from_peak = 'M182T541', end_peak = 'M262T526'), and retrieveSubNetwork3(base_peaks = c('M182T541', 'M262T526'), base_adducts = c('[M+H]+', '[M+H]+')). The README notes 'the folder_output should keep same among different layer subnetworks'. Finally apply mergeSubnetwork(from_peak = 'M182T541', end_peak = 'M262T526'), which consolidates the layers by retaining the larger subnetwork when conflicting base peaks arise and merging redundant ion-form peaks of the same metabolite — the mechanism by which conflicting annotations (e.g., M153T555 / 5-ureido-4-imidazole vs. xanthosine M285T555) resolve toward maximum subnetwork self-consistency.

## Related tools

- **MetDNA2Vis** (R package that reconstructs the three KGMN network layers, extracts layer subnetworks, and merges them via reconstructNetwork1-3, retrieveSubNetwork1-3, and mergeSubnetwork) — https://github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (R package installed alongside MetDNA2Vis to support spectral handling in the workflow) — https://github.com/ZhuMetLab/SpectraTools

## Examples

```
mergeSubnetwork(from_peak = 'M182T541', end_peak = 'M262T526', folder_output = 'M182T541_M262T526')
```

## Evaluation signals

- All three layer subnetworks were written into the same folder_output ('M182T541_M262T526') with no path mismatches
- After mergeSubnetwork, the merged subnetwork exists in the shared output folder and the three layers have resolved into one consolidated subnetwork
- No remaining conflicting base-peak duplicates in the merged subnetwork; redundant ion-form peaks of the same metabolite are merged
- The reconstruction used the exact parameters from the tutorial (is_unknown_annotation = TRUE, centric_met c('C00082','KeggExd000923'), from_peak/end_peak M182T541/M262T526, base_adducts '[M+H]+')
- Conflicting annotations resolve toward maximum subnetwork self-consistency, as in the reported xanthosine (M285T555) case

## Limitations

- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers)
- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy
- Removing spectrally similar but hard-to-interpret connections loses interpretive information present in data-driven molecular networks

## Evidence

- [readme] readme workflow step: "Note: the folder_output should keep same among different layer subnetworks"
- [readme] reconstructNetwork1 call: "reconstructNetwork1(is_unknown_annotation = TRUE)"
- [methods] retrieveSubNetwork1 signature: "retrieveSubNetwork1(centric_met = c('C00082', 'KeggExd000923'), is_unknown_annotation = TRUE"
- [abstract] three-layer integration: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [results] conflict resolution finding: [evidence span withheld — anchor status: not_found]
