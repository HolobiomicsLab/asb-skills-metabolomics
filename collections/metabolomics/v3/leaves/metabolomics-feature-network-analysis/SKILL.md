---
name: metabolomics-feature-network-analysis
description: Use when you have a list of statistically significant LC-MS features
  (m/z, retention time) selected by MB-VIP permutation testing (p < 0.01) and need
  to group them into isotopologue, adduct, cross-assay and correlation clusters to
  prioritize cross-assay biomarkers.
when_to_use_negative:
- The input is raw LC-MS vendor data (e.g. mzML/NetCDF) rather than a peak-picked
  feature table with m/z and retention time per feature — MamsiStructSearch consumes
  feature matrices, not raw spectra.
- Features have not been statistically filtered (no MB-VIP permutation p-values);
  the structural search is designed for selected significant features, not full feature
  lists.
- The assay was not analysed by the National Phenome Centre and you require automated
  annotation — the README notes annotation is 'only supported for assays analysed
  by the National Phenome Centre'.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MAMSI (mamsi.mamsi_struct_search.MamsiStructSearch)
  role: 'Core engine: performs RT-window isotopologue search, adduct neutral-mass
    matching, cross-assay linking, correlation clustering and network visualization
    of significant features'
  repo: https://github.com/kopeckylukas/py-mamsi
- name: networkx
  role: Builds and exports the structural feature network as an NX object for inspection
    in Cytoscape
  repo: https://github.com/kopeckylukas/py-mamsi
- name: pyvis
  role: Renders the interactive network plot with node colours per flattened correlation
    cluster and structural edges
  repo: https://github.com/kopeckylukas/py-mamsi
- name: peakPantheR ROI files (npc-open-lcms)
  role: Provides annotated metabolite retention time and m/z values enabling automated
    putative annotation within MamsiStructSearch
  repo: https://github.com/phenomecentre/npc-open-lcms
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:10f71a90560c04bcba964cf5798fc8b3d73dc49104dcec220c84df5f2a1a7828
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - MAMSI (mamsi.mamsi_struct_search.MamsiStructSearch)
  - Python
  - joblib
  - mamsi
  - matplotlib
  - mbpls
  - networkx
  - numpy
  - pandas
  - peakPantheR ROI files (npc-open-lcms)
  - pyvis
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 24
---

# metabolomics-feature-network-analysis

## Summary

Groups statistically significant untargeted LC-MS metabolomics features into clusters defined by structural properties (isotopologues, adducts, cross-assay links) based on m/z and retention time, and visualizes them as an interactive network. In the MAMSI workflow this step turns MB-VIP/permtest feature hits into interpretable structural relationships across assays.

## When to use

Use when you have a list of statistically significant LC-MS features (m/z, retention time) selected by MB-VIP permutation testing (p < 0.01) and need to group them into isotopologue, adduct, cross-assay and correlation clusters to prioritize cross-assay biomarkers.

## When NOT to use

- The input is raw LC-MS vendor data (e.g. mzML/NetCDF) rather than a peak-picked feature table with m/z and retention time per feature — MamsiStructSearch consumes feature matrices, not raw spectra.
- Features have not been statistically filtered (no MB-VIP permutation p-values); the structural search is designed for selected significant features, not full feature lists.
- The assay was not analysed by the National Phenome Centre and you require automated annotation — the README notes annotation is 'only supported for assays analysed by the National Phenome Centre'.

## Inputs

- Pandas DataFrame of statistically significant LC-MS features (concatenated multi-assay feature matrix with assay-prefixed m/z and RT columns), selected at p < 0.01 from mb_vip_permtest
- Assay-prefixed feature tables (e.g. alz_hpos.csv, alz_lpos.csv, alz_lneg.csv style with HPOS_/LPOS_/LNEG_ prefixes)
- Optional peakPantheR ROI files (retention time and m/z annotations) from npc-open-lcms for automated annotation

## Outputs

- Structural clusters (isotopologue, adduct, merged, and cross-assay groups) assigned to significant features
- Correlation clusters flattened via silhouette method
- Network plot (static or interactive via pyvis) and an optional NetworkX graph object exportable to Cytoscape
- Putative annotations for features measured in supported NPC chromatography assays

## How to apply

Feed the selected significant feature table (columns prefixed per assay, e.g. HPOS_/LPOS_/LNEG_) into MamsiStructSearch via load_lcms after instantiating it with rt_win=5 (seconds RT windows) and ppm=10 (m/z tolerance). The tool first splits features into 5-second RT windows, searches for isotopologue signatures via 1.00335 Da mass differences, then matches hypothetical neutral masses from common electrospray adducts (15 ppm tolerance in the described search) to group adduct clusters; overlapping isotopologue and adduct clusters are merged into structural clusters. Cross-assay clusters are found using [M+H]+/[M-H]- as link references, and get_correlation_clusters with flat_method='silhouette' flattens hierarchical correlation structure (e.g. max_clusters=11). Finally, get_structural_network draws the network where node colours represent flattened correlation clusters and edges encode structural links (adduct, isotopologue, cross-assay); it can be returned as an NX object for export to Cytoscape. Optional automated annotation via annotate=True uses peakPantheR ROI files and is only supported for National Phenome Centre assays.

## Related tools

- **MAMSI (mamsi.mamsi_struct_search.MamsiStructSearch)** (Core engine: performs RT-window isotopologue search, adduct neutral-mass matching, cross-assay linking, correlation clustering and network visualization of significant features) — https://github.com/kopeckylukas/py-mamsi
- **networkx** (Builds and exports the structural feature network as an NX object for inspection in Cytoscape) — https://github.com/kopeckylukas/py-mamsi
- **pyvis** (Renders the interactive network plot with node colours per flattened correlation cluster and structural edges) — https://github.com/kopeckylukas/py-mamsi
- **peakPantheR ROI files (npc-open-lcms)** (Provides annotated metabolite retention time and m/z values enabling automated putative annotation within MamsiStructSearch) — https://github.com/phenomecentre/npc-open-lcms

## Examples

```
from mamsi.mamsi_struct_search import MamsiStructSearch; struct = MamsiStructSearch(rt_win=5, ppm=10); struct.load_lcms(selected); struct.get_structural_clusters(annotate=True)
```

## Evaluation signals

- Isotopologue groups correspond to features whose m/z differences equal 1.00335 Da within the configured 10 ppm tolerance and fall inside the same 5-second RT window.
- Adduct clusters share a common hypothetical neutral mass within the predefined tolerance (15 ppm) after correcting for common electrospray adducts.
- Cross-assay clusters link features across HPOS/LPOS/LNEG blocks via [M+H]+/[M-H]- reference pairs.
- Flattened correlation clustering succeeds without degenerate output (silhouette method returns a sensible number of clusters, e.g. the max_clusters=11 used in the tutorial).
- The returned NetworkX object contains node colours matching flattened correlation clusters and edges typed as adduct, isotopologue, or cross-assay links, and can be opened in Cytoscape.

## Limitations

- Automated annotation of untargeted LC-MS features is only supported for assays analysed by the National Phenome Centre (via peakPantheR ROI files).
- Cluster links are structural correlations, not confirmed identities — the paper reports putative annotation only, with majority agreement of cross-assay biomarker signs of association against literature.
- The framework was tested on metabolomics phenotyping data; the README notes it 'should be usable with other types of LC-MS data' but this is not guaranteed.

## Evidence

- [intro] linking statistically significant features of untargeted multi-assay liquid chromatography – mass spectrometry (LC-MS) metabolomics datasets into clusters defined by their structural properties: "linking statistically significant features of untargeted multi-assay liquid chromatography – mass spectrometry (LC-MS) metabolomics datasets into clusters defined by their structural properties"
- [readme] all features are split into retention time (*RT*) windows of 5 seconds intervals, then each RT window is searched for isotopologue signatures by searching mass differences of 1.00335 Da: "all features are split into retention time (*RT*) windows of 5 seconds intervals, then each RT window is searched for isotopologue signatures by searching mass differences of 1.00335 Da"
- [readme] If hypothetical neutral masses match for two or more features within a pre-defined tolerance (15 *ppm*) then these features are grouped together.: "If hypothetical neutral masses match for two or more features within a pre-defined tolerance (15 *ppm*) then these features are grouped together."
- [methods] struct = MamsiStructSearch(rt_win=5, ppm=10): "struct = MamsiStructSearch(rt_win=5, ppm=10)"
- [readme] Annotation of untargeted LC-MS features (only supported for assays analysed by the National Phenome Centre).: "Annotation of untargeted LC-MS features (only supported for assays analysed by the National Phenome Centre)."
