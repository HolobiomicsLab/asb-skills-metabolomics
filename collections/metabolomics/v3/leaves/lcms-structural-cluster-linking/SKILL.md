---
name: lcms-structural-cluster-linking
description: Use when you have statistically significant features from untargeted
  multi-assay LC-MS metabolomics datasets and want to group them into structural clusters
  based on retention time and mass-to-charge ratio, enabling putative annotation.
when_to_use_negative:
- The input feature table has not been reduced to statistically significant features
  (e.g. full unfiltered intensity matrix) — structural search is meant to operate
  on the p<0.01 selection from mb_vip_permtest
- The data are single-assay only with no expectation of cross-assay structural links
  (M+H]+/[M-H]-), since MamsiStructSearch is designed for multi-assay LC-MS integration
- The analyte platform is not LC-MS (e.g. GC-MS, NMR, flow-injection), as the RT-window
  and electrospray adduct logic does not apply
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: mamsi
  role: Python package providing MamsiStructSearch, the class implementing RT-window
    split, isotopologue/adduct search, correlation clustering, network visualisation,
    and peakPantheR-based annotation
  repo: https://github.com/kopeckylukas/py-mamsi
- name: Python
  role: Runtime environment (>=3.9) with pandas/numpy used to load and concatenate
    assay feature tables and slice the significant-feature selection
- name: pyvis
  role: Dependency used to render the interactive structural network from get_structural_network()
- name: networkx
  role: Dependency backing the NX network object returned by get_structural_network()
    for export to Cytoscape
- name: peakPantheR
  role: NPC open-LCMS ROI files consumed by MamsiStructSearch's annotate=True path
    for automated feature annotation
  repo: https://github.com/phenomecentre/npc-open-lcms
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:ebf6e63b42568cb9f8ccc509c281b8b07859ed4daa25c307a261648b806af6b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - Python
  - joblib
  - mamsi
  - matplotlib
  - mbpls
  - networkx
  - numpy
  - pandas
  - peakPantheR
  - pyvis
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# lcms-structural-cluster-linking

## Summary

Links statistically significant LC-MS features from multi-assay metabolomics data into clusters defined by structural properties (m/z, retention time), adduct/isotopologue relationships, and cross-assay links using MamsiStructSearch.

## When to use

Use when you have statistically significant features from untargeted multi-assay LC-MS metabolomics datasets and want to group them into structural clusters based on retention time and mass-to-charge ratio, enabling putative annotation.

## When NOT to use

- The input feature table has not been reduced to statistically significant features (e.g. full unfiltered intensity matrix) — structural search is meant to operate on the p<0.01 selection from mb_vip_permtest
- The data are single-assay only with no expectation of cross-assay structural links (M+H]+/[M-H]-), since MamsiStructSearch is designed for multi-assay LC-MS integration
- The analyte platform is not LC-MS (e.g. GC-MS, NMR, flow-injection), as the RT-window and electrospray adduct logic does not apply

## Inputs

- Pandas DataFrame of statistically significant features (rows = samples, columns = features) with assay-prefixed column names (e.g. HPOS_, LPOS_, LNEG_)
- Selected feature subset from np.where(p_vals < 0.01) applied to MB-VIP permutation p-values
- Optional: NPC ROI annotation files from peakPantheR for automated putative annotation

## Outputs

- Structural cluster table grouping features by isotopologue, adduct, and merged structural clusters
- Correlation cluster assignments (flattened hierarchical clusters via silhouette)
- Network graph (pyvis interactive plot or NetworkX NX object) of structural relationships between features
- Putative metabolite annotations for supported NPC LC-MS assays

## How to apply

After MB-PLS and MB-VIP permutation testing selects significant features (p < 0.01), pass them to MamsiStructSearch(rt_win=5, ppm=10). Call .load_lcms() on the concatenated feature table, then .get_structural_clusters(annotate=True) which first splits features into 5-second RT windows, searches for isotopologue signatures via mass differences of 1.00335 Da between m/z values, then groups features sharing hypothetical neutral masses derived from common electrospray adducts within the ppm tolerance; overlapping isotopologue and adduct clusters are merged. Optionally run .get_correlation_clusters(flat_method='silhouette', max_clusters=11) to add correlation-based grouping, and .get_structural_network() to visualize/serialize the links (e.g., as an NX object for Cytoscape). The annotate=True path uses ROI files from peakPantheR (phenomecentre/npc-open-lcms) for automated annotation based on chromatographic RT and m/z.

## Related tools

- **mamsi** (Python package providing MamsiStructSearch, the class implementing RT-window split, isotopologue/adduct search, correlation clustering, network visualisation, and peakPantheR-based annotation) — https://github.com/kopeckylukas/py-mamsi
- **Python** (Runtime environment (>=3.9) with pandas/numpy used to load and concatenate assay feature tables and slice the significant-feature selection)
- **pyvis** (Dependency used to render the interactive structural network from get_structural_network())
- **networkx** (Dependency backing the NX network object returned by get_structural_network() for export to Cytoscape)
- **peakPantheR** (NPC open-LCMS ROI files consumed by MamsiStructSearch's annotate=True path for automated feature annotation) — https://github.com/phenomecentre/npc-open-lcms

## Examples

```
struct = MamsiStructSearch(rt_win=5, ppm=10); struct.load_lcms(selected); struct.get_structural_clusters(annotate=True)
```

## Evaluation signals

- Each retained feature must appear in at least one structural cluster, correlation cluster, or singleton row in the output table
- Isotopologue groupings should show mass differences of ~1.00335 Da within a 5 s RT window, and adduct groups should agree on hypothetical neutral mass within the ppm tolerance (10–15 ppm)
- Cross-assay links must be anchored on [M+H]+ / [M-H]- reference pairs between prefixed assay blocks (HPOS_/LPOS_/LNEG_)
- The returned network (NX object) should contain only the selected significant features as nodes, with edges labelled by structural link type (adduct, isotopologue, cross-assay)
- With annotate=True, features matched to NPC ROI files should carry consistent retention time and m/z against the NPC chromatographic method

## Limitations

- Automated annotation is only supported for assays analysed by the National Phenome Centre (ROI files); other platforms fall back to structural clustering without annotation
- ppm tolerance (10–15) and rt_win (5 s) are user-tunable; too-loose tolerance can merge unrelated features, too-tight may split true isotopologues
- Correlation clustering depends on the chosen flat_method and max_clusters (e.g. silhouette, max_clusters=11), so group counts can vary between runs/parameterisations
- Putative annotations remain hypothesis-level and must be reconciled against literature-reported bilirubin/biliverdin ion species or similar references

## Evidence

- [methods] structural search parameters: [evidence span withheld — anchor status: not_found]
- [readme] isotopologue + adduct search logic: "searched for isotopologue signatures by searching mass differences of 1.00335 Da between mass-to-charge ratios (*m/z*) of the features"
- [readme] adduct tolerance: "If hypothetical neutral masses match for two or more features within a pre-defined tolerance (15 *ppm*) then these features are grouped together"
- [readme] correlation clustering: "struct.get_correlation_clusters(flat_method='silhouette', max_clusters=11)"
- [readme] network output: "network = struct.get_structural_network(include_all=True, interactive=False, labels=True, return_nx_object=True)"
