---
name: lcms-isotopologue-pattern-detection
description: Use when you have a table of significant features whose column headers
  encode AssayName_RTsec_m/z, and you need to group isotopologues and adducts into
  structural clusters across multi-assay LC-MS metabolomics data.
when_to_use_negative:
- The feature columns do not encode retention time and m/z in the AssayName_RTsec_m/z
  header format, so load_lcms() cannot parse the structural metadata
- Features have not yet been filtered by statistical significance (e.g. MB-VIP permutation
  p-values < 0.01), since the structural search is designed to operate on the selected
  significant-feature subset
- The data are not untargeted LC-MS metabolomics features with electrospray-ionisation
  adduct behaviour (the framework was tested on metabolomics phenotyping data)
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: mamsi
  role: Provides MamsiStructSearch, whose load_lcms(), get_structural_clusters() methods
    perform the RT-windowed 1.00335 Da isotopologue search and cluster merging
  repo: https://github.com/kopeckylukas/py-mamsi
- name: pandas
  role: Holds the significant-feature intensity table passed to load_lcms() after
    column-prefix concatenation (e.g. pd.concat([hpos, lpos, lneg], axis=1))
- name: npc-open-lcms
  role: Supplies ROI/annotation files used by the structural search tool for automated
    annotation of features based on RT for a given chromatography and m/z
  repo: https://github.com/phenomecentre/npc-open-lcms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:91a0c2360d2f435ddb71bd89ab5995a49e156aa13dc2ff7bf30da5d5d13f42c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - github.com/phenomecentre/npc-open-lcms
  - mamsi
  - matplotlib
  - npc-open-lcms
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# lcms-isotopologue-pattern-detection

## Summary

Detects isotopologue signatures among statistically significant LC-MS features by searching for 1.00335 Da mass differences between m/z values within 5-second retention-time windows, as implemented in MAMSI's MamsiStructSearch. This groups features that plausibly derive from the same metabolite and underpins subsequent adduct and structural-cluster merging.

## When to use

Use when you have a table of significant features whose column headers encode AssayName_RTsec_m/z, and you need to group isotopologues and adducts into structural clusters across multi-assay LC-MS metabolomics data.

## When NOT to use

- The feature columns do not encode retention time and m/z in the AssayName_RTsec_m/z header format, so load_lcms() cannot parse the structural metadata
- Features have not yet been filtered by statistical significance (e.g. MB-VIP permutation p-values < 0.01), since the structural search is designed to operate on the selected significant-feature subset
- The data are not untargeted LC-MS metabolomics features with electrospray-ionisation adduct behaviour (the framework was tested on metabolomics phenotyping data)

## Inputs

- Intensity table of statistically significant LC-MS features (selected by MB-VIP permutation test, p < 0.01), with column headers in AssayName_RTsec_m/z format (e.g. HPOS_, LPOS_, LNEG_ prefixed columns)
- Tolerance parameters: retention-time window in seconds (e.g. rt_win=5) and mass tolerance in ppm (e.g. ppm=10 or 15)

## Outputs

- Isotopologue group and Isotopologue pattern assignments for each input feature
- Structural clusters formed by merging overlapping isotopologue and adduct clusters
- Cross-assay links between clusters via [M+H]+/[M-H]- references
- Feature network edges encoding isotopologue links (weight 1)

## How to apply

Load the significant-feature intensity table with MamsiStructSearch.load_lcms(), which parses assay name, retention time (seconds), and m/z from the AssayName_RTsec_m/z column-header format. Instantiate MamsiStructSearch with RT and ppm tolerances (README example: rt_win=5, ppm=10; the article also mentions a 15 ppm tolerance). All features are first split into 5-second RT windows, and each window is searched for isotopologue signatures as mass differences of 1.00335 Da between feature m/z values; two or more such features are grouped as an isotopologue cluster. Overlapping isotopologue and adduct clusters (the latter from hypothetical neutral masses of common electrospray adducts within the ppm tolerance) are merged into structural clusters, and cross-assay links are found using [M+H]+/[M-H]- as reference masses. This isotopologue grouping step is the basis for downstream correlation-cluster comparison (get_correlation_clusters) and network visualisation (get_structural_network).

## Related tools

- **mamsi** (Provides MamsiStructSearch, whose load_lcms(), get_structural_clusters() methods perform the RT-windowed 1.00335 Da isotopologue search and cluster merging) — https://github.com/kopeckylukas/py-mamsi
- **pandas** (Holds the significant-feature intensity table passed to load_lcms() after column-prefix concatenation (e.g. pd.concat([hpos, lpos, lneg], axis=1)))
- **npc-open-lcms** (Supplies ROI/annotation files used by the structural search tool for automated annotation of features based on RT for a given chromatography and m/z) — https://github.com/phenomecentre/npc-open-lcms

## Examples

```
struct = MamsiStructSearch(rt_win=5, ppm=10); struct.load_lcms(selected); struct.get_structural_clusters(adducts='all', annotate=True)
```

## Evaluation signals

- Every input feature must receive an assigned Isotopologue group and Isotopologue pattern column in the structural-group membership table
- Features grouped as isotopologues must differ in m/z by approximately 1.00335 Da and fall within the same 5-second RT window
- Overlapping isotopologue and adduct clusters must be merged so that each feature has a single Structural cluster assignment
- The network graph must contain an edge for every structurally linked feature pair, with isotopologue links weighted 1
- Cross-assay links must connect clusters across assays through matching [M+H]+/[M-H]- reference masses within the ppm tolerance

## Limitations

- The search uses fixed tolerances (e.g. rt_win=5 seconds, 10–15 ppm), so features with RT drift or larger mass error may be missed or wrongly grouped
- Isotopologue grouping is putative: it identifies signatures by mass difference and RT co-elution, not confirmed metabolite identity
- Automated annotation is only supported for assays analysed by the National Phenome Centre, via ROI files from peakPantheR

## Evidence

- [methods] Isotopologue search via 1.00335 Da RT-windowed mass differences: "all features are split into retention time (RT) windows of 5 seconds intervals, then each RT window is searched for isotopologue signatures by searching mass differences of 1.00335 Da"
- [readme] MamsiStructSearch instantiation and cluster call: "struct = MamsiStructSearch(rt_win=5, ppm=10)
struct.load_lcms(selected)
struct.get_structural_clusters(annotate=True)"
- [intro] Structural clustering purpose of the MAMSI framework: "linking statistically significant features of untargeted multi-assay liquid chromatography – mass spectrometry (LC-MS) metabolomics datasets into clusters defined by their structural properties based"
- [readme] Merging overlapping clusters into structural clusters: "Overlapping adduct clusters and isotopologue clusters are then merged to form structural clusters. Further, we search cross-assay clusters using [M+H]+/[M-H]- as link references."
