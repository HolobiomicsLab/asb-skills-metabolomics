---
name: putative-metabolite-annotation
description: Use when statistically significant LC-MS features (e.g. MB-VIP permutation
  p < 0.01) must be assigned putative metabolite identities by grouping them into
  structural clusters (m/z/RT) and matching to known adducts, isotopologues, and fragments
  in HMDB or literature.
when_to_use_negative:
- Features have not yet been filtered for statistical significance (e.g. no MB-VIP
  permutation p-value threshold applied) — annotation should follow feature selection
- Assays were not analysed by the National Phenome Centre platform — README states
  automated feature annotation is 'only supported for assays analysed by the National
  Phenome Centre'
- The goal is definitive metabolite identification (MS/MS confirmation) rather than
  putative annotation based on m/z and RT
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: MAMSI (mamsi)
  role: Provides MamsiStructSearch for structural clustering (rt_win, ppm), automated
    annotation via ROI files, and correlation clustering of significant LC-MS features
  repo: https://github.com/kopeckylukas/py-mamsi
- name: npc-open-lcms
  role: Supplies ROI files with annotated metabolite RT and m/z values used by MamsiStructSearch's
    annotate=True option
  repo: https://github.com/phenomecentre/npc-open-lcms
- name: peakPantheR
  role: Underlying targeted extraction framework whose ROI annotation files drive
    the automated putative annotation step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:f1101a68625fe7745d3fb6f2ca9e8f9e7d1784291397041b55dfa74383e51c69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - MAMSI (mamsi)
  - Python
  - mamsi
  - matplotlib
  - mbpls
  - npc-open-lcms
  - numpy
  - pandas
  - peakPantheR
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 20
---

# putative-metabolite-annotation

## Summary

Putative annotation of statistically significant untargeted LC-MS features from multi-assay metabolomics data, using MAMSI's structural search (m/z and retention time based clustering) and ROI-based annotation, then matching adducts, isotopologues, and fragments to known ions (e.g. via HMDB) and literature.

## When to use

Use when statistically significant LC-MS features (e.g. MB-VIP permutation p < 0.01) must be assigned putative metabolite identities by grouping them into structural clusters (m/z/RT) and matching to known adducts, isotopologues, and fragments in HMDB or literature.

## When NOT to use

- Features have not yet been filtered for statistical significance (e.g. no MB-VIP permutation p-value threshold applied) — annotation should follow feature selection
- Assays were not analysed by the National Phenome Centre platform — README states automated feature annotation is 'only supported for assays analysed by the National Phenome Centre'
- The goal is definitive metabolite identification (MS/MS confirmation) rather than putative annotation based on m/z and RT

## Inputs

- Feature table of statistically significant untargeted LC-MS features (e.g. selected via MB-VIP permutation p < 0.01), with columns prefixed per assay (e.g. HPOS_, LPOS_, LNEG_)
- Per-feature m/z and retention time values from LC-MS intensity tables
- ROI files with metabolite annotations (RT and m/z) for the NPC LC-MS assays (github.com/phenomecentre/npc-open-lcms)
- Optional external annotation resource (e.g. HMDB) and literature references for candidate metabolite ions

## Outputs

- Structural clusters of significant features (adduct clusters, isotopologue clusters, merged structural groups, cross-assay [M+H]+/[M-H]- links)
- Correlation clusters (flattened, e.g. via 'silhouette' method with max_clusters parameter)
- Putative annotations per feature (e.g. 15 of 18 bilirubin-signature features linked to bilirubin adducts/isotopologues/fragments, one to biliverdin)
- Network object of structural relationships (NX object exportable to Cytoscape)

## How to apply

After feature selection, load the significant features into MamsiStructSearch (e.g. rt_win=5, ppm=15, or ppm=10 as in the README quickstart) via .load_lcms(). Structural links are found by splitting features into 5-second retention time windows, detecting isotopologue signatures via mass differences of 1.00335 Da, then computing hypothetical neutral masses for common electrospray adducts and grouping features matching within 15 ppm; overlapping adduct and isotopologue clusters are merged, and cross-assay clusters are linked using [M+H]+/[M-H]- references. Run .get_structural_clusters(annotate=True) to obtain structural groups with automated annotation based on RT and m/z using peakPantheR ROI files, and .get_correlation_clusters() for correlation grouping. Finally, compare the resulting structural/correlation clusters against known ion identities (e.g. bilirubin adducts, isotopologues and fragments in HMDB) and prior literature to assign putative annotations, checking signs of association where models are directional.

## Related tools

- **MAMSI (mamsi)** (Provides MamsiStructSearch for structural clustering (rt_win, ppm), automated annotation via ROI files, and correlation clustering of significant LC-MS features) — https://github.com/kopeckylukas/py-mamsi
- **npc-open-lcms** (Supplies ROI files with annotated metabolite RT and m/z values used by MamsiStructSearch's annotate=True option) — https://github.com/phenomecentre/npc-open-lcms
- **peakPantheR** (Underlying targeted extraction framework whose ROI annotation files drive the automated putative annotation step)

## Examples

```
struct = MamsiStructSearch(rt_win=5, ppm=15); struct.load_lcms(selected); struct.get_structural_clusters(annotate=True); struct.get_correlation_clusters()
```

## Evaluation signals

- Structural clusters are internally consistent: isotopologue links show ~1.00335 Da mass differences within 5 s RT windows; adduct links give matching hypothetical neutral masses within the ppm tolerance (10–15 ppm)
- Cross-assay links reconcile via [M+H]+/[M-H]- reference pairs across assay blocks
- Reproducing the reference task: 18 significant bilirubin-signature features resolve into two structural groups and three singletons, with 15 features linked to bilirubin-related ions and one to biliverdin
- Majority of putative cross-assay biomarkers and their signs of association agree with previously reported literature results
- Network visualisation (structural network with flattened correlation clusters as node colours) shows expected adduct/isotopologue/cross-assay edges

## Limitations

- Automated annotation via get_structural_clusters(annotate=True) is only supported for assays analysed by the National Phenome Centre; other data require manual annotation
- Annotation is putative, based on m/z, RT, and adduct/isotopologue logic — not definitive identification
- Results depend on the chosen tolerances (rt_win=5 s, ppm=10 or 15) and correlation clustering settings (e.g. flat_method='silhouette', max_clusters=11)

## Evidence

- [readme] ROI annotation: [evidence span withheld — anchor status: not_found]
- [readme] structural search parameters: "searched for isotopologue signatures by searching mass differences of 1.00335 Da between mass-to-charge ratios (m/z) of the features"
- [readme] annotation-only-for-NPC: "Annotation of untargeted LC-MS features (only supported for assays analysed by the National Phenome Centre)."
- [abstract] literature agreement: "Following putative annotation, the majority of these and their signs of association agreed with results previously reported in the literature."
