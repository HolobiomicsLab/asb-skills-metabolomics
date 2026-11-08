---
name: metabolomics-standard-library-matching
description: Use when untargeted LC-MS/MS peak tables with MS2 spectra (per ionization
  mode) need seed annotations of known metabolites to launch knowledge-guided multi-layer
  network (KGMN) propagation to unknowns.
when_to_use_negative:
- The goal is annotating unknowns with no library entry at all — library matching
  alone cannot assign them; proceed to KGMN reaction/MS2-similarity propagation instead
- Input is raw vendor instrument files without a processed peak table and MS2 spectra
  — peak detection must precede library matching
- Samples from non-model organisms or exposomics profiles, where organism-specific
  spectral libraries and reaction knowledge are not yet incorporated
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MetDNA2Vis
  role: Provides the cpd_lib standard spectral library and the matching workflow that
    seeds known metabolite annotations from MS1 m/z and MS2 spectra
  repo: github.com/ZhuMetLab/MetDNA2Vis
- name: SpectraTools
  role: Companion ZhuMetLab package installed alongside MetDNA2Vis for spectral handling
    during library matching
  repo: ZhuMetLab/SpectraTools
- name: igraph
  role: CRAN dependency used by MetDNA2Vis for network representation once seeded
    annotations feed into the multi-layer network
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34537-6
    title: Metabolite annotation from knowns to unknowns through knowledge-guided
      multi-layer metabolic networking
  source_content_hash: sha256:6939ebb4662d588b1664271575531efdff614a407227b41a75159ac99648c17f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetDNA2Vis
  - SpectraTools
  - igraph
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# metabolomics-standard-library-matching

## Summary

Seeds KGMN annotation by matching precursor-ion MS1 m/z and MS2 spectra of LC-MS/MS peaks against a packaged standard spectral library (cpd_lib) via MetDNA2Vis, distinguishing known metabolites from unknown peaks per ionization mode (e.g., 39 known in positive and 41 known in negative mode for the 46std_mix + human liver S9 incubation).

## When to use

Use when untargeted LC-MS/MS peak tables with MS2 spectra (per ionization mode) need seed annotations of known metabolites to launch knowledge-guided multi-layer network (KGMN) propagation to unknowns.

## When NOT to use

- The goal is annotating unknowns with no library entry at all — library matching alone cannot assign them; proceed to KGMN reaction/MS2-similarity propagation instead
- Input is raw vendor instrument files without a processed peak table and MS2 spectra — peak detection must precede library matching
- Samples from non-model organisms or exposomics profiles, where organism-specific spectral libraries and reaction knowledge are not yet incorporated

## Inputs

- LC-MS/MS MS1 peak table (per ionization mode, e.g., 700 peaks positive / 741 peaks negative for the 46std_mix S9 incubation)
- MS2 tandem mass spectra for the detected peaks
- Standard spectral library cpd_lib (packaged with MetDNA2Vis)

## Outputs

- Seed annotation table of library-matched known metabolites per ionization mode (39 known positive, 41 known negative for the reference incubation)
- Annotation table for downstream KGMN network construction, via reformatTable1()

## How to apply

Load the peak table and MS2 spectra (e.g., the 46std_mix human liver S9 incubation data) into R with MetDNA2Vis and SpectraTools installed from GitHub plus dplyr, tidyr, readr, CHNOSZ, igraph, magrittr, ggplot2, ggraph, and tidygraph. Match each peak's MS1 m/z (exact precursor mass) and MS2 spectrum against the packaged standard spectral library cpd_lib from MetDNA2Vis to identify known metabolites present after incubation. Run this independently per ionization mode, since known counts differ (39 known of 700 MS1 peaks in positive mode; 41 known of 741 peaks in negative mode in the published incubation). These library-matched knowns become the seed set for the downstream reaction network (reconstructNetwork1 with is_unknown_annotation = TRUE) and knowledge-guided MS2 similarity network (reconstructNetwork2 after reformatTable1). Peaks without a confident library match remain candidates for reaction-paired unknown annotation rather than being discarded.

## Related tools

- **MetDNA2Vis** (Provides the cpd_lib standard spectral library and the matching workflow that seeds known metabolite annotations from MS1 m/z and MS2 spectra) — github.com/ZhuMetLab/MetDNA2Vis
- **SpectraTools** (Companion ZhuMetLab package installed alongside MetDNA2Vis for spectral handling during library matching) — ZhuMetLab/SpectraTools
- **igraph** (CRAN dependency used by MetDNA2Vis for network representation once seeded annotations feed into the multi-layer network)

## Examples

```
library(MetDNA2Vis); library(SpectraTools); reconstructNetwork1(is_unknown_annotation = TRUE); annotation_table <- reformatTable1()
```

## Evaluation signals

- Positive-mode output must annotate exactly 39 known peaks out of 700 MS1 peaks; negative mode exactly 41 known of 741 peaks for the 46std_mix S9 incubation
- Annotated entries must include the IMP-derived cascade products inosine and inosine 5'-sulfate, which were confirmed with authentic chemical standards
- Every seed annotation must be supported by agreement of both precursor MS1 m/z and MS2 spectrum against cpd_lib, not m/z alone
- Resulting annotation table must be accepted by reformatTable1() and feed reconstructNetwork2 without errors
- Known-metabolite identification accuracy in the multilayer strategy should reach >95% per the published evaluation

## Limitations

- The multilayer strategy including library seeding cannot effectively discriminate structurally similar isomers of unknown metabolites (e.g., isocitrate glucuronidation product isomers)
- Validating unknown annotations remains a grand challenge because unknowns are absent from spectral databases and lack chemical standards
- ~40% of reaction-paired metabolites have low MS/MS similarity scores despite high structural similarity, limiting downstream annotation accuracy even after correct seed matching

## Evidence

- [intro] Seed annotation by MS1/MS2 library matching: "the most commonly used approach is to search the exact mass of precursor ion (MS1 m/z) and tandem mass spectrum (MS2 spectrum) against standard spectral libraries"
- [readme] Workflow placement of library matching before network construction: "annotation_table <- reformatTable1()
reconstructNetwork2(annotation_table = annotation_table)"
- [methods] Network 1 extends knowns to reaction-paired unknowns: "reconstructNetwork1(is_unknown_annotation = TRUE)"
- [full_text] Reference counts per ionization mode for the incubation: [evidence span withheld — anchor status: not_found]
