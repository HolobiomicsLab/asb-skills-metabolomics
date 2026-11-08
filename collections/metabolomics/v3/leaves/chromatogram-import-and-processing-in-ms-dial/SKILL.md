---
name: chromatogram-import-and-processing-in-ms-dial
description: Use when raw centroid-mode LC–MS all-ion-fragmentation chromatograms
  need feature detection and pseudo-MS/MS deconvolution to produce a comparable annotation
  baseline (e.g. against MetaboAnnotatoR rank-1 or manual expert lists) on datasets
  like Lipid+, Lipid−, or HILIC+.
when_to_use_negative:
- When the goal is to annotate only a statistically pre-selected subset of significant
  features rather than all detected features, since MS-DIAL performs annotation in
  an untargeted manner and such a selection is currently not possible
- When chromatograms are in profile (non-centroid) mode, as the workflow requires
  raw LC–MS AIF data acquired/transformed in centroid mode
- When targeted MS/MS fragmentation experiments have already been acquired and a dedicated
  DDA spectral matching workflow is preferred
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MS-DIAL
  role: Primary software used to import AIF chromatograms, detect features, run MS2Dec
    deconvolution, and perform untargeted spectral library annotation
- name: LipidBlast
  role: Default in-silico lipidomics library (>1 million spectra) searched against
    deconvoluted pseudo-MS/MS spectra for lipid annotation
- name: METLIN
  role: Public MS/MS spectral library used alongside MassBank and GNPS to annotate
    features from the pseudo-MS/MS spectra
- name: MassBank
  role: Public MS/MS spectral library searched against deconvoluted spectra for feature
    annotation
- name: GNPS
  role: Public MS/MS spectral library searched against deconvoluted spectra for feature
    annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c03032
    title: Automated Annotation of Untargeted All-Ion Fragmentation LC–MS Metabolomics
      Data with MetaboAnnotatoR
  source_content_hash: sha256:3cb35eb3a1572b776c9e9a5b454a094558f0f92f0a1a89a13007b2e6316ddb11
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - LipidBlast
  - METLIN
  - MS-DIAL
  - MassBank
  - MetaboAnnotatoR
  - R
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# Chromatogram import and processing in MS-DIAL

## Summary

Imports raw centroid-mode LC–MS AIF chromatograms into MS-DIAL 4.38, runs feature detection, and applies MS2Dec deconvolution to reconstruct pseudo-MS/MS spectra by correlating co-eluting fragment peaks with parent ions, producing annotated feature lists for benchmark comparison against other annotation workflows.

## When to use

Use when raw centroid-mode LC–MS all-ion-fragmentation chromatograms need feature detection and pseudo-MS/MS deconvolution to produce a comparable annotation baseline (e.g. against MetaboAnnotatoR rank-1 or manual expert lists) on datasets like Lipid+, Lipid−, or HILIC+.

## When NOT to use

- When the goal is to annotate only a statistically pre-selected subset of significant features rather than all detected features, since MS-DIAL performs annotation in an untargeted manner and such a selection is currently not possible
- When chromatograms are in profile (non-centroid) mode, as the workflow requires raw LC–MS AIF data acquired/transformed in centroid mode
- When targeted MS/MS fragmentation experiments have already been acquired and a dedicated DDA spectral matching workflow is preferred

## Inputs

- Raw centroid-mode LC–MS AIF chromatograms (e.g. MESA AIF QC runs for Lipid+, Lipid−, HILIC+ datasets)
- In-silico and public spectral libraries: LipidBlast, METLIN, MassBank, GNPS

## Outputs

- Detected feature list with retention times and m/z from MS-DIAL feature detection
- Deconvoluted pseudo-MS/MS spectra per feature from MS2Dec
- Untargeted annotation table (feature → spectral library match) for all detected features

## How to apply

Import the raw LC–MS AIF chromatograms (acquired/transformed in centroid mode) into MS-DIAL 4.38 and run its feature detection to obtain detected features across the dataset. Apply the MS2Dec deconvolution routine in MS-DIAL, which reconstructs pseudo-MS/MS spectra by inspecting which fragment peaks co-elute with parent ions, then search the resulting spectra against libraries such as LipidBlast (in-silico lipid library, >1 million spectra) and public MS/MS libraries (METLIN, MassBank, GNPS) to obtain untargeted annotations for all detected features. Because MS-DIAL performs annotation in an untargeted manner and does not support selecting a subset of features prior to annotation, the full feature list is annotated by default; intersect these lists afterwards with manual expert annotations or another tool's rank-1 annotations to compute overlap counts and percentages. Validate the resulting feature and annotation lists against a reference set (e.g. 161 manually annotated features for Lipid+, 53 for Lipid−, 50 for HILIC+) before comparing across tools.

## Related tools

- **MS-DIAL** (Primary software used to import AIF chromatograms, detect features, run MS2Dec deconvolution, and perform untargeted spectral library annotation)
- **LipidBlast** (Default in-silico lipidomics library (>1 million spectra) searched against deconvoluted pseudo-MS/MS spectra for lipid annotation)
- **METLIN** (Public MS/MS spectral library used alongside MassBank and GNPS to annotate features from the pseudo-MS/MS spectra)
- **MassBank** (Public MS/MS spectral library searched against deconvoluted spectra for feature annotation)
- **GNPS** (Public MS/MS spectral library searched against deconvoluted spectra for feature annotation)

## Examples

```
MS-DIAL 4.38 → import centroid-mode AIF chromatogram (Lipid+ QC) → run feature detection → run MS2Dec deconvolution → search LipidBlast/METLIN/MassBank/GNPS
```

## Evaluation signals

- Detected feature count matches the dataset's known feature count (e.g. 161 manually annotated features for Lipid+, 53 for Lipid−, 50 for HILIC+) so overlap percentages are computed on the correct denominators
- Manual-vs-MS-DIAL overlap counts reproduce the reference figures: 89/161 (55%) for Lipid+, 39/53 (74%) for Lipid−, 31/50 (62%) for HILIC+
- MS2Dec deconvolution succeeds for the majority of parent ions, evidenced by non-empty pseudo-MS/MS spectra for features included in the annotation lists
- Annotation table covers all detected features (untargeted), not a filtered subset
- Spectral library search results fall within the expected mass accuracy of the instrument (quadrupole time-of-flight) and report plausible adducts and isotopologues

## Limitations

- Peak shape matching and correlation deconvolution strategies can result in incomplete pseudo-MS/MS spectra, particularly for co-eluting compounds with similar chemical groups producing nonspecific fragment ions or for poorly detected fragment peaks
- In MS-DIAL, feature selection is currently not possible and annotations are performed in an untargeted manner, which can be time-consuming and may miss low-intensity features
- Isotopologues, in-source fragments, and some adducts are not always considered by existing open-source tools for AIF LC–MS annotation
- Low recall for non-lipid metabolites is attributable to the small size of nonlipid fragment libraries

## Evidence

- [intro] MS2Dec deconvolution reconstructs pseudo-MS/MS spectra by correlating co-eluting fragment peaks: "such as LipidMatch,19 R-MetaboList,9,20 or MS-DIAL,7 the parent–fragment
ion relationship is
determined by inspecting which fragment peaks co-elute with the parent
ions"
- [intro] MS-DIAL performs untargeted annotation and does not support pre-selecting a feature subset: "In MS-DIAL, such a selection
is currently not possible, and the annotations are performed in an
untargeted manner"
- [intro] Deconvolution strategies can produce incomplete pseudo-MS/MS spectra for co-eluting or poorly detected fragments: "These can
result in incomplete pseudo-MS/MS spectra, particularly for co-eluting
compounds containing similar types of chemical groups, which upon
fragmentation produce nonspecific fragment ions"
- [intro] LipidBlast contains over 1 million in-silico spectra used as default lipidomics library: "The default lipidomics databases (LipidBlast35) containing over 1 million spectra"
- [readme] Raw AIF chromatograms must be in centroid mode: "It requires raw LC-MS AIF chromatograms acquired/transformed in centroid mode."
- [results] Reference overlap figures for benchmarking against manual annotations: "MetaboAnnotatoR achieved a better overlap with manual annotations than MS-DIAL for Lipid+ [121/161 (75%) vs 89/161 (55%)]"
