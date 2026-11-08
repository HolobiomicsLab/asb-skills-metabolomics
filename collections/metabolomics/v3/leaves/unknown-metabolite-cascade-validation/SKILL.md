---
name: unknown-metabolite-cascade-validation
description: Use when LC-MS/MS peak tables and MS2 spectra from an S9 incubation (or
  similar in vitro enzymatic reaction) need KGMN unknown-peak annotations reproduced
  and validated, e.g. expected 39+69/700 pos and 41+106/741 neg peaks.
when_to_use_negative:
- Non-model organisms or environmental/exposomics profiles, since organism-specific
  databases are not yet incorporated and caution is advised
- Distinguishing structurally similar isomers of unknowns (e.g., isocitrate glucuronidation
  product isomers), which KGMN cannot discriminate effectively
- Cases where the input is already a fully annotated feature table with no unknown
  peaks requiring cascade annotation
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MetDNA2Vis
  role: Implements reconstructNetwork1/2/3, reformatTable1, retrieveSubNetwork*, and
    mergeSubnetwork to build, merge, and visualize the KGMN layers that propagate
    annotations from known seeds to unknown peaks
  repo: https://github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Dependency for spectral handling when matching MS2 spectra against the cpd_lib
    standard spectral library
  repo: https://github.com/ZhuMetLab/SpectraTools
- name: igraph
  role: CRAN dependency underlying network construction and manipulation in the KGMN
    layers
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:9e486fbb74b59243374f6502e30412e9c531ea952ab46fc2a0ec257b684b3a3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuMetLab/MetDNA2Vis
  tools:
  - MetDNA2Vis
  - SpectraTools
  - igraph
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# unknown-metabolite-cascade-validation

## Summary

Validates KGMN-annotated unknown metabolites in the 46std_mix human liver S9 incubation by reproducing the known-to-unknown annotation cascade per ionization mode and confirming IMP-derived unknowns (inosine and inosine 5'-sulfate) with authentic chemical standards.

## When to use

Use when LC-MS/MS peak tables and MS2 spectra from an S9 incubation (or similar in vitro enzymatic reaction) need KGMN unknown-peak annotations reproduced and validated, e.g. expected 39+69/700 pos and 41+106/741 neg peaks.

## When NOT to use

- Non-model organisms or environmental/exposomics profiles, since organism-specific databases are not yet incorporated and caution is advised
- Distinguishing structurally similar isomers of unknowns (e.g., isocitrate glucuronidation product isomers), which KGMN cannot discriminate effectively
- Cases where the input is already a fully annotated feature table with no unknown peaks requiring cascade annotation

## Inputs

- 46std_mix + human liver S9 incubation peak table with MS1 m/z and intensities per ionization mode (700 MS1 peaks positive, 741 negative)
- MS2 spectra for the S9-incubation peaks
- Packaged standard spectral library (cpd_lib from MetDNA2Vis)
- Metabolite annotation table produced by reformatTable1()

## Outputs

- Per-mode annotation summary: 39 known + 69 unknown of 700 MS1 peaks (positive); 41 known + 106 unknown of 741 peaks (negative)
- Merged three-layer annotation CSV assigning known/unknown labels to each peak
- Subnetwork visualizations of the IMP-derived cascade (inosine, inosine 5'-sulfate) with reaction and MS2-similarity edges
- Validated annotations of IMP-derived unknowns confirmed by chemical standards

## How to apply

Load the 46std_mix S9-incubation peak table and MS2 spectra, then seed annotation by matching MS1 m/z and MS2 spectra against the packaged cpd_lib standard spectral library with MetDNA2Vis (39 knowns in positive mode, 41 in negative mode). Build the knowledge-based metabolic reaction network layer with reconstructNetwork1(is_unknown_annotation = TRUE) to extend knowns to reaction-paired unknown peaks, prepare the annotation table with reformatTable1(), and add the knowledge-guided MS2 similarity layer via reconstructNetwork2(annotation_table = annotation_table) so annotations propagate from known seeds to spectrally similar reaction-paired unknowns. Construct the global peak correlation layer with reconstructNetwork3(), merge all three layers, and label remaining unassigned peaks as unknowns. Trace the IMP-derived cascade to inosine and inosine 5'-sulfate through reaction and MS2-similarity edges, then validate those two unknowns against authentic chemical standards.

## Related tools

- **MetDNA2Vis** (Implements reconstructNetwork1/2/3, reformatTable1, retrieveSubNetwork*, and mergeSubnetwork to build, merge, and visualize the KGMN layers that propagate annotations from known seeds to unknown peaks) — https://github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Dependency for spectral handling when matching MS2 spectra against the cpd_lib standard spectral library) — https://github.com/ZhuMetLab/SpectraTools
- **igraph** (CRAN dependency underlying network construction and manipulation in the KGMN layers)

## Examples

```
library(MetDNA2Vis); annotation_table <- reformatTable1(); reconstructNetwork1(is_unknown_annotation = TRUE); reconstructNetwork2(annotation_table = annotation_table); reconstructNetwork3()
```

## Evaluation signals

- Output CSV reports exactly 39 known + 69 unknown of 700 MS1 peaks in positive mode and 41 known + 106 unknown of 741 peaks in negative mode
- Merged annotation table contains entries for the IMP cascade products inosine and inosine 5'-sulfate
- Subnetwork exports for M182T541_M262T526 exist from all three layers (retrieveSubNetwork1/2/3) in the same folder_output before mergeSubnetwork succeeds
- Annotation accuracy of known seeds consistent with the reported >95% identification accuracy of the multilayer strategy
- IMP-derived unknown annotations are corroborated by matching authentic chemical standards

## Limitations

- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting annotation accuracy
- KGMN cannot effectively discriminate structurally similar isomers of unknown metabolites
- Structural accuracy of unknowns depends on in silico biotransformation algorithms (BioTransformer reaction sets mainly originate from human metabolic reactions)
- Only five recurrent unknown metabolites were validated with synthetic standards via repository-mining; unknowns absent from spectral databases and lacking standards are hard to validate

## Evidence

- [abstract] Three-layer KGMN integration: "The KGMN approach integrates three-layer networks, including knowledge-based metabolic reaction network, knowledge-guided MS/MS similarity network, and global peak correlation network."
- [methods] Unknown-annotation network construction: "reconstructNetwork1(is_unknown_annotation = TRUE)"
- [methods] MS2 similarity layer: "annotation_table <- reformatTable1()"
- [readme] Subnetwork merge: "mergeSubnetwork(from_peak = 'M182T541', 
  end_peak = 'M262T526', 
  folder_output = 'M182T541_M262T526')"
- [discussion] Isomer discrimination limitation: "isocitrate has four reactive functional groups for glucuronidation, whose product isomers cannot be distinguished effectively in KGMN"
