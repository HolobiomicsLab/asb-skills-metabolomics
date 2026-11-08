---
name: unknown-peak-annotation-consolidation
description: Use when a metabolic feature peak (e.g. M153T555, M182T541) is claimed
  by multiple overlapping KGMN subnetworks with conflicting annotations or base peaks,
  and annotations must be merged or pruned into one consolidated subnetwork.
when_to_use_negative:
- When constructing the global networks themselves rather than extracting/merging
  subnetworks from an already-built KGMN result
- When the feature is claimed by only a single non-overlapping subnetwork, so no consolidation
  decision is needed
- When distinguishing structurally similar isomers is the goal — KGMN cannot effectively
  discriminate such isomers (e.g. isocitrate glucuronidation product isomers)
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetDNA2Vis
  role: R package providing retrieveSubNetwork1/2/3 and mergeSubnetwork to extract
    and consolidate the three KGMN subnetwork layers
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Companion ZhuMetLab R package installed alongside MetDNA2Vis for spectra handling
    during subnetwork retrieval and merging
  repo: https://github.com/ZhuMetLab/SpectraTools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:b63cfcec0700d0a3e79cf358750c2e07fe63a5e1b85cafd87da9d71bf8246c58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - MetDNA2Vis
  - SpectraTools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# unknown-peak-annotation-consolidation

## Summary

Consolidates overlapping KGMN peak-annotation subnetworks (metabolic-reaction, peak-to-peak, MS2-similarity layers) by merging redundant ion-form peaks and retaining the larger self-consistent subnetwork when conflicting base peaks arise, resolving claims of a metabolic feature by multiple subnetworks.

## When to use

Use when a metabolic feature peak (e.g. M153T555, M182T541) is claimed by multiple overlapping KGMN subnetworks with conflicting annotations or base peaks, and annotations must be merged or pruned into one consolidated subnetwork.

## When NOT to use

- When constructing the global networks themselves rather than extracting/merging subnetworks from an already-built KGMN result
- When the feature is claimed by only a single non-overlapping subnetwork, so no consolidation decision is needed
- When distinguishing structurally similar isomers is the goal — KGMN cannot effectively discriminate such isomers (e.g. isocitrate glucuronidation product isomers)

## Inputs

- KGMN annotation_table (from reformatTable1) with peak IDs like M182T541, M262T526, M285T555
- Global layer-1 (metabolic reaction), layer-2 (peak-to-peak), and layer-3 (MS2-similarity) networks from reconstructNetwork1/2/3
- Centric metabolite IDs (KEGG C00082, extended KeggExd000923) and base-peak/adduct pairs (e.g. [M+H]+)
- Demo dataset: Demo_MetDNA2_NIST_urine_pos/06_visualization working directory

## Outputs

- Consolidated subnetwork per folder_output (e.g. 'M182T541_M262T526') with all three layers merged
- A single annotation per feature after resolving conflicting base-peak duplicates and redundant ion-form peaks

## How to apply

Load MetDNA2Vis and SpectraTools, set the working directory to the project's 06_visualization folder, and first construct the three global networks via reconstructNetwork1 (is_unknown_annotation = TRUE), reformatTable1/reconstructNetwork2, and reconstructNetwork3. Extract the three layer-1/2/3 subnetworks with retrieveSubNetwork1 (centric_met, is_unknown_annotation), retrieveSubNetwork2 (from_peak, end_peak), and retrieveSubNetwork3 (base_peaks, base_adducts), keeping folder_output identical (e.g. 'M182T541_M262T526') across layers. Then run mergeSubnetwork with from_peak and end_peak on the same folder_output to consolidate: retain the larger subnetwork when conflicting base peaks arise and merge redundant ion-form peaks of the same metabolite, maximizing subnetwork self-consistency. In the reported conflict, the subnetwork of M153T555 annotated as 5-ureido-4-imidazole was removed so all peak annotations merged under xanthosine (M285T555), later confirmed as an in-source fragment by chemical standard.

## Related tools

- **MetDNA2Vis** (R package providing retrieveSubNetwork1/2/3 and mergeSubnetwork to extract and consolidate the three KGMN subnetwork layers) — https://github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Companion ZhuMetLab R package installed alongside MetDNA2Vis for spectra handling during subnetwork retrieval and merging) — https://github.com/ZhuMetLab/SpectraTools

## Examples

```
library(MetDNA2Vis); mergeSubnetwork(from_peak = 'M182T541', end_peak = 'M262T526', folder_output = 'M182T541_M262T526')
```

## Evaluation signals

- The merged subnetwork exists in the shared folder_output 'M182T541_M262T526' after mergeSubnetwork
- All three layers have resolved into one consolidated subnetwork with no remaining conflicting base-peak duplicates
- Redundant ion-form peaks of the same metabolite (e.g. M285T555 xanthosine and its in-source fragment) carry a single consistent annotation
- checkFiles4Vis(dir_path) reports all expected visualization files present for the merged subnetwork
- Rule check (empirical rules) passes per subnetwork during retrieveSubNetwork1 extraction

## Limitations

- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers)
- KGMN removes spectrally similar but hard-to-interpret connections, losing interpretive information present in data-driven molecular networks
- Validating unknown annotations remains a grand challenge because unknowns are absent from spectral databases and lack chemical standards

## Evidence

- [methods] mergeSubnetwork consolidation rule: "mergeSubnetwork(from_peak = 'M182T541', end_peak = 'M262T526'"
- [abstract] three-layer network integration: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [readme] README retrieval/merge workflow: "# merge subnetwork
mergeSubnetwork(from_peak = 'M182T541', 
  end_peak = 'M262T526', 
  folder_output = 'M182T541_M262T526')"
- [readme] same folder_output across layers: "# Note: the folder_output should keep same among different layer subnetworks"
- [discussion] isomer discrimination limitation: "isocitrate has four reactive functional groups for glucuronidation, whose product isomers cannot be distinguished effectively in KGMN"
