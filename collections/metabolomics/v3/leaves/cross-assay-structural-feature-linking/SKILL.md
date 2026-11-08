---
name: cross-assay-structural-feature-linking
description: Use when a significant-feature intensity table (one column per feature,
  headers encoding assay, RT in seconds and m/z) needs grouping into isotopologue,
  adduct, and cross-assay structural clusters, and network edges encoding those links
  are required.
when_to_use_negative:
- The input is a full (unfiltered) peak-intensity table rather than the reduced set
  of statistically significant features — the structural search is designed to interpret
  MB-VIP permutation-test selections, not raw feature lists
- Features come from non-LC-MS or non-ESI data where RT/m/z parsing and the adduct-neutral-mass
  assumptions (15 ppm in the paper's quickstart; 10 ppm in the code example) do not
  apply
- Only a single assay is available and no cross-assay linking is desired — the pipeline
  is built around multi-assay LC-MS integration (e.g. HPOS/LPOS/LNEG)
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: mamsi
  role: Provides MamsiStructSearch, which loads the significant-feature table, performs
    the RT-window/m/z structural search, correlation clustering, and network construction
  repo: https://github.com/kopeckylukas/py-mamsi
- name: MAMSI
  role: The overall framework in which structural linking of significant features
    is implemented
  repo: https://github.com/kopeckylukas/py-mamsi
- name: pandas
  role: Used to load and concatenate per-assay feature tables (e.g. HPOS_, LPOS_,
    LNEG_ prefixed CSVs) that become the input to load_lcms()
- name: networkx
  role: Underlies get_structural_network() for building and returning the structural-links
    graph
- name: npc-open-lcms
  role: Supplies ROI files of metabolite annotations (RT and m/z for NPC LC-MS assays)
    enabling automated feature annotation during the structural search
  repo: https://github.com/phenomecentre/npc-open-lcms
- name: scikit-learn
  role: Used in get_correlation_clusters() for hierarchical clustering and silhouette-based
    flattening of correlation clusters
- name: matplotlib
  role: Rendering the non-interactive structural network plot when interactive=False
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:3e885b1cf99da98a5ee9254fd8fdf0092f41aba6dbbf172caf5a63f20305a8d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - MAMSI
  - mamsi
  - matplotlib
  - networkx
  - npc-open-lcms
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# Cross-assay structural feature linking

## Summary

Groups statistically significant untargeted LC-MS features into clusters defined by structural properties — retention time, m/z differences (isotopologues), common adducts, and cross-assay [M+H]+/[M-H]- links — using the MAMSI structural-search pipeline.

## When to use

Use when a significant-feature intensity table (one column per feature, headers encoding assay, RT in seconds and m/z) needs grouping into isotopologue, adduct, and cross-assay structural clusters, and network edges encoding those links are required.

## When NOT to use

- The input is a full (unfiltered) peak-intensity table rather than the reduced set of statistically significant features — the structural search is designed to interpret MB-VIP permutation-test selections, not raw feature lists
- Features come from non-LC-MS or non-ESI data where RT/m/z parsing and the adduct-neutral-mass assumptions (15 ppm in the paper's quickstart; 10 ppm in the code example) do not apply
- Only a single assay is available and no cross-assay linking is desired — the pipeline is built around multi-assay LC-MS integration (e.g. HPOS/LPOS/LNEG)

## Inputs

- Pandas DataFrame of statistically significant feature intensities (one column per feature), column headers encoding assay name, retention time (seconds) and m/z (e.g. HPOS_123_145.0789)
- Optional ROI annotation files from peakPantheR (github.com/phenomecentre/npc-open-lcms) for automated annotation of features by RT and m/z

## Outputs

- Structural-group membership table with per-feature columns: Isotopologue group, Isotopologue pattern, Adduct group, Adduct, Structural cluster, Correlation cluster, Cross-assay link
- NetworkX graph of structurally linked feature pairs, with node colours reflecting flattened hierarchical correlation clusters and edge weights encoding link type
- Optional interactive network plot (pyvis)

## How to apply

Load the significant-feature table (the output of MB-VIP permutation testing with p<0.01 in the AddNeuroMed classification workflow) with MamsiStructSearch.load_lcms(), parsing the column header format AssayName_RTsec_m/z. Instantiate MamsiStructSearch with rt_win=5 seconds and ppm=10 so features are first partitioned into 5-second RT windows, then searched for isotopologue signatures via 1.00335 Da mass differences within each window. Search for common electrospray adducts by computing hypothetical neutral masses and grouping features whose neutral masses match within the 10 ppm tolerance; overlapping isotopologue and adduct clusters are merged into structural clusters, and cross-assay links are found using [M+H]+/[M-H]- as link references. Optionally call get_correlation_clusters() with flat_method='silhouette' to cluster features hierarchically on their correlations, and build a network with get_structural_network() where edges encode isotopologue (weight 1), adduct (weight 5) and cross-assay (weight 10) links.

## Related tools

- **mamsi** (Provides MamsiStructSearch, which loads the significant-feature table, performs the RT-window/m/z structural search, correlation clustering, and network construction) — https://github.com/kopeckylukas/py-mamsi
- **MAMSI** (The overall framework in which structural linking of significant features is implemented) — https://github.com/kopeckylukas/py-mamsi
- **pandas** (Used to load and concatenate per-assay feature tables (e.g. HPOS_, LPOS_, LNEG_ prefixed CSVs) that become the input to load_lcms())
- **networkx** (Underlies get_structural_network() for building and returning the structural-links graph)
- **npc-open-lcms** (Supplies ROI files of metabolite annotations (RT and m/z for NPC LC-MS assays) enabling automated feature annotation during the structural search) — https://github.com/phenomecentre/npc-open-lcms
- **scikit-learn** (Used in get_correlation_clusters() for hierarchical clustering and silhouette-based flattening of correlation clusters)
- **matplotlib** (Rendering the non-interactive structural network plot when interactive=False)

## Examples

```
struct = MamsiStructSearch(rt_win=5, ppm=10); struct.load_lcms(selected); struct.get_structural_clusters(annotate=True)
```

## Evaluation signals

- The structural-group membership table must contain, for every input feature, assigned Isotopologue group, Isotopologue pattern, Adduct group, Adduct, Structural cluster, Correlation cluster and Cross-assay link columns
- Every structurally linked feature pair must appear as an edge in the network graph returned by get_structural_network(return_nx_object=True)
- Isotopologue groups should differ by ~1.00335 Da in m/z and fall within the same 5-second RT window
- Adduct-group members must share a hypothetical neutral mass within the configured ppm tolerance (10 ppm in the quickstart; 15 ppm described in the paper text)
- Cross-assay links should connect pairs whose [M+H]+/[M-H]- references are consistent (edge weight 10)

## Limitations

- Automated annotation of untargeted features is supported only for assays analysed by the National Phenome Centre (via NPC ROI files); other chromatography/assay sources are not annotated
- The framework was tested on metabolomics phenotyping data; although the README notes it should be usable with other LC-MS data, behaviour elsewhere is unverified
- Accuracy of isotopologue/adduct assignment depends on header parsing of the AssayName_RTsec_m/z format and on the chosen rt_win and ppm tolerances; mis-parsed or shifted RT values will break 5-second windowing

## Evidence

- [intro] structural clustering method: "linking statistically significant features of untargeted multi-assay liquid chromatography – mass spectrometry (LC-MS) metabolomics datasets into clusters defined by their structural properties based"
- [methods] structural search description: "all features are split into retention time (RT) windows of 5 seconds intervals, then each RT window is searched for isotopologue signatures by searching mass differences of 1.00335 Da"
- [methods] adduct and cross-assay search: "This is achieved by calculating hypothetical neutral masses based on common adducts in electrospray ionisation. If hypothetical neutral masses match for two or more features within a pre-defined"
- [methods] cross-assay linking references: [evidence span withheld — anchor status: not_found]
- [readme] README invocation snippet: "struct = MamsiStructSearch(rt_win=5, ppm=10)
struct.load_lcms(selected)
struct.get_structural_clusters(annotate=True)"
- [readme] README network snippet: "network = struct.get_structural_network(include_all=True, interactive=False, labels=True, return_nx_object=True)"
