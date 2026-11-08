---
name: adduct-group-mass-difference-search
description: Use when a significant-feature intensity table with columns named as
  AssayName_RTsec_m/z is available after MB-PLS significance testing (e.
when_to_use_negative:
- The input table is a full untargeted feature table (e.g. all LC-MS peaks before
  MB-PLS significance testing) rather than a subset of statistically significant features
  — applying structural search on non-significant features is out of scope and dilutes
  the analysis.
- The feature column headers do not encode assay name, RT, and m/z in the AssayName_RTsec_m/z
  format — the search cannot parse the features without this naming convention.
- The data is GC-MS or another chromatography technique without m/z differences interpretable
  as adduct/isotopologue relationships — MAMSI was tested on LC-MS metabolomics phenotyping
  data.
edam_operation: http://edamontology.org/operation_3636
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MAMSI (mamsi)
  role: Provides the MamsiStructSearch class used to load the significant-feature
    table and run the structural search (isotopologue, adduct, cross-assay clustering
    and network building).
  repo: github.com/kopeckylukas/py-mamsi
- name: peakPantheR
  role: Optional source of region-of-interest (ROI) annotation files used by MamsiStructSearch
    for automated annotation of features based on RT for a given chromatography and
    m/z.
  repo: github.com/phenomecentre/npc-open-lcms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:03b163b5f3ee53c9ac2f9715585497eea841e97c17e299feaa68da30d8c07203
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAMSI (mamsi)
  - mamsi
  - matplotlib
  - numpy
  - pandas
  - peakPantheR
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3636
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# adduct-group-mass-difference-search

## Summary

Link statistically significant LC-MS metabolomics features into structural clusters by detecting isotopologue signatures (1.00335 Da mass differences), common electrospray adducts (hypothetical neutral mass matching within ppm tolerance), and cross-assay [M+H]+/[M-H]- links, using the MAMSI MamsiStructSearch tool.

## When to use

Use when a significant-feature intensity table with columns named as AssayName_RTsec_m/z is available after MB-PLS significance testing (e.g. empirical p-values from mb_vip_permtest) and you want structural annotation, grouping and cross-assay linking of those features by retention time and mass-to-charge ratio.

## When NOT to use

- The input table is a full untargeted feature table (e.g. all LC-MS peaks before MB-PLS significance testing) rather than a subset of statistically significant features — applying structural search on non-significant features is out of scope and dilutes the analysis.
- The feature column headers do not encode assay name, RT, and m/z in the AssayName_RTsec_m/z format — the search cannot parse the features without this naming convention.
- The data is GC-MS or another chromatography technique without m/z differences interpretable as adduct/isotopologue relationships — MAMSI was tested on LC-MS metabolomics phenotyping data.

## Inputs

- Significant-feature intensity table with column headers of the form AssayName_RTsec_m/z (e.g. HPOS_123_150.1234), with one column per feature.
- Optional peakPantheR ROI annotation files from the NPC Open LC-MS platform for automated annotation of features by RT and m/z.

## Outputs

- Structural-group membership table with columns: Isotopologue group, Isotopologue pattern, Adduct group, Adduct, Structural cluster, Correlation cluster, Cross-assay link.
- Network graph (NX object) with edges encoding isotopologue links (weight 1), adduct links (weight 5), and cross-assay links (weight 10).

## How to apply

1. Load the significant-feature table with MamsiStructSearch.load_lcms(), parsing assay name, retention time (seconds), and m/z from the column headers. 2. Instantiate MamsiStructSearch with tolerances appropriate to the data (e.g. rt_win=5 seconds, ppm=10). 3. Run get_structural_clusters(adducts='all', annotate=True) to find isotopologue signatures via 1.00335 Da mass differences within the RT window, adduct clusters via hypothetical neutral mass matching within the ppm tolerance, merge overlapping isotopologue and adduct clusters into structural clusters, and link structural clusters across assays using [M+H]+/[M-H]- as references. 4. Validate that the structural-group membership table contains Isotopologue group, Isotopologue pattern, Adduct group, Adduct, Structural cluster, Correlation cluster, and Cross-assay link columns for every input feature.

## Related tools

- **MAMSI (mamsi)** (Provides the MamsiStructSearch class used to load the significant-feature table and run the structural search (isotopologue, adduct, cross-assay clustering and network building).) — github.com/kopeckylukas/py-mamsi
- **peakPantheR** (Optional source of region-of-interest (ROI) annotation files used by MamsiStructSearch for automated annotation of features based on RT for a given chromatography and m/z.) — github.com/phenomecentre/npc-open-lcms

## Examples

```
from mamsi.mamsi_struct_search import MamsiStructSearch; struct = MamsiStructSearch(rt_win=5, ppm=10); struct.load_lcms(selected); struct.get_structural_clusters(adducts='all', annotate=True)
```

## Evaluation signals

- The structural-group membership table contains, for every input feature, the assigned Isotopologue group, Isotopologue pattern, Adduct group, Adduct, Structural cluster, Correlation cluster, and Cross-assay link columns.
- The network graph contains an edge for every structurally linked feature pair (isotopologue, adduct, or cross-assay link).
- Every isotopologue link reflects a 1.00335 Da m/z difference within a 5-second RT window.
- Every adduct link reflects a hypothetical neutral mass match within the configured ppm tolerance (e.g. 10 ppm).
- Every cross-assay link is anchored on a [M+H]+/[M-H]- reference pair.

## Limitations

- The search relies on the column-header naming convention AssayName_RTsec_m/z; features not following this format cannot be parsed.
- Automated annotation is only supported for assays analysed by the National Phenome Centre (requires NPC ROI files).
- Adduct matching uses a fixed ppm tolerance (10 in the quickstart) and isotopologue detection uses a fixed 1.00335 Da difference within a 5-second RT window — parameters may need adjustment for other chromatographies or instruments.
- The framework was tested on metabolomics phenotyping data; other LC-MS data types may require validation.

## Evidence

- [methods] isotopologue signatures via 1.00335 Da mass differences within 5-second RT windows: "then each RT window is searched for isotopologue signatures by searching mass differences of 1.00335 Da between mass-to-charge ratios (m/z) of the features"
- [methods] hypothetical neutral mass matching within ppm tolerance: "This is achieved by calculating hypothetical neutral masses based on common adducts in electrospray ionisation. If hypothetical neutral masses match for two or more features within a pre-defined"
- [methods] structural clusters, cross-assay linking and annotation: "Overlapping adduct clusters and isotopologue clusters are then merged to form structural clusters. Further, we search cross-assay clusters using [M+H]+/[M-H]- as link references."
- [readme] network edges encode structural link types: [evidence span withheld — anchor status: external_doc]
- [readme] scope caveat for non-LC-MS data: "*N.B. the framework was tested on metabolomics phenotyping data, but it should be usable with other types of LC-MS data.*"
