---
name: ms2-adduct-base-peak-specification
description: Use when extracting a layer-3 MS2-similarity subnetwork with retrieveSubNetwork3
  and you must declare which peaks serve as base peaks and their adduct identities
  before merging subnetworks.
when_to_use_negative:
- When the analysis goal is layer-1 or layer-2 extraction (retrieveSubNetwork1/2 take
  centric metabolites or peak pairs, not base_peaks/base_adducts)
- When the sample contains no confirmed adduct assignment for the candidate base peak
  — adducts must be known (e.g., '[M+H]+') rather than guessed
- When annotating structurally similar isomers of unknown metabolites, which KGMN
  cannot effectively discriminate
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MetDNA2Vis
  role: Provides retrieveSubNetwork3 (base_peaks/base_adducts arguments), reconstructNetwork1-3,
    and mergeSubnetwork for extracting and consolidating the three-layer KGMN subnetworks
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Companion R package installed alongside MetDNA2Vis for spectral handling during
    MS2-similarity subnetwork extraction
  repo: github.com/ZhuMetLab/SpectraTools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:92185d4eed772ec6b2a8787e930936885a54924831d4b8ff7cda37d57face048
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetDNA2Vis
  - SpectraTools
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# ms2-adduct-base-peak-specification

## Summary

Specifies base peaks and their adduct forms (e.g., '[M+H]+') when extracting the layer-3 MS2-similarity subnetwork in the KGMN/MetDNA2Vis workflow, so that MS2 spectral similarity edges are anchored to the correct ion-form representatives of each metabolite.

## When to use

Use when extracting a layer-3 MS2-similarity subnetwork with retrieveSubNetwork3 and you must declare which peaks serve as base peaks and their adduct identities before merging subnetworks.

## When NOT to use

- When the analysis goal is layer-1 or layer-2 extraction (retrieveSubNetwork1/2 take centric metabolites or peak pairs, not base_peaks/base_adducts)
- When the sample contains no confirmed adduct assignment for the candidate base peak — adducts must be known (e.g., '[M+H]+') rather than guessed
- When annotating structurally similar isomers of unknown metabolites, which KGMN cannot effectively discriminate

## Inputs

- Peak identifiers from the MetDNA2 annotation table (e.g., 'M182T541', 'M262T526')
- Adduct annotations for the base peaks (e.g., '[M+H]+')
- Demo_MetDNA2_NIST_urine_pos/06_visualization working directory with MetDNA2 output files (checked via checkFiles4Vis)
- Same folder_output string used for layer-1 and layer-2 subnetwork retrieval

## Outputs

- Layer-3 MS2-similarity subnetwork files written to folder_output 'M182T541_M262T526'
- Merged subnetwork after mergeSubnetwork with conflicting base-peak duplicates resolved

## How to apply

After retrieving the layer-1 metabolic-reaction-network subnetwork (retrieveSubNetwork1, e.g., centric_met = c('C00082', 'KeggExd000923'), is_unknown_annotation = TRUE) and the layer-2 peak-to-peak subnetwork (retrieveSubNetwork2 with from_peak/end_peak such as 'M182T541' and 'M262T526'), call retrieveSubNetwork3 with base_peaks = c('M182T541', 'M262T526') and base_adducts = c('[M+H]+', '[M+H]+'), keeping folder_output identical across all three layers ('M182T541_M262T526') as the README requires. The base peaks and adducts define which ion forms anchor the MS2-similarity network, so they must match the peaks used in the layer-2 call. During mergeSubnetwork consolidation, retain the larger subnetwork when conflicting base peaks arise and merge redundant ion-form peaks of the same metabolite; correct base-peak/adduct specification is what allows conflicting annotations to resolve (e.g., M153T555's 5-ureido-4-imidazole annotation being removed so all peaks merge under xanthosine M285T555, confirmed as an in-source fragment by chemical standard).

## Related tools

- **MetDNA2Vis** (Provides retrieveSubNetwork3 (base_peaks/base_adducts arguments), reconstructNetwork1-3, and mergeSubnetwork for extracting and consolidating the three-layer KGMN subnetworks) — github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Companion R package installed alongside MetDNA2Vis for spectral handling during MS2-similarity subnetwork extraction) — github.com/ZhuMetLab/SpectraTools

## Examples

```
retrieveSubNetwork3(base_peaks = c('M182T541', 'M262T526'), base_adducts = c('[M+H]+', '[M+H]+'), folder_output = 'M182T541_M262T526')
```

## Evaluation signals

- The merged subnetwork exists in the shared folder_output 'M182T541_M262T526' and all three layers have resolved into one consolidated subnetwork with no remaining conflicting base-peak duplicates
- folder_output string is identical across retrieveSubNetwork1/2/3 and mergeSubnetwork calls (README explicitly notes it should stay the same)
- Base adducts passed to retrieveSubNetwork3 correspond one-to-one with base_peaks entries (e.g., two peaks, two '[M+H]+' entries)
- Redundant ion-form peaks of the same metabolite are merged rather than duplicated in the consolidated subnetwork
- checkFiles4Vis(dir_path) passes on the 06_visualization directory before subnetwork retrieval

## Limitations

- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers), so base-peak anchoring cannot resolve isomer ambiguity
- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy in the layer-3 network
- KGMN removes some spectrally similar but hard-to-interpret connections, losing interpretive information present in data-driven molecular networks

## Evidence

- [readme] retrieveSubNetwork3 signature: "retrieveSubNetwork3(base_peaks = c('M182T541', 'M262T526'),
  base_adducts = c('[M+H]+', '[M+H]+'),
  folder_output = c('M182T541_M262T526'))"
- [readme] Note on consistent folder_output: "# Note: the folder_output should keep same among different layer subnetworks"
- [readme] merge subnetwork step: "# merge subnetwork
mergeSubnetwork(from_peak = 'M182T541', 
  end_peak = 'M262T526', 
  folder_output = 'M182T541_M262T526')"
- [abstract] three-layer integration: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [results] conflict resolution finding: [evidence span withheld — anchor status: not_found]
