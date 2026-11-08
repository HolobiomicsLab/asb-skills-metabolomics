---
name: grouped-bar-abundance-visualization
description: Use when you have a superclass-by-organism relative abundance table from
  ConCISE consensus annotations and need a grouped bar chart to visually compare chemical-class
  composition across exudate source organisms.
when_to_use_negative:
- The annotation output has not been aggregated to superclass-level relative abundances
  per organism (e.g., you only have raw per-feature annotations or class/subclass-level
  tables).
- The analysis target is a single organism or sample set with no grouping variable
  to compare across bar groups.
- You need statistical testing of class-level compositional differences rather than
  a descriptive visualization.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3520
tools:
- name: ConCISE
  role: Upstream tool producing the consensus annotation file whose superclass labels
    are aggregated and visualized
  repo: https://github.com/Zquinlan/conCISE
- name: Python
  role: Used to compute superclass relative abundances and render the grouped bar
    chart in matplotlib
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo12121275
    title: 'ConCISE: Consensus Annotation Propagation of Ion Features in Untargeted
      Tandem Mass Spectrometry Combining Molecular Networking and In Silico Metabolite
      Structure Prediction'
  source_content_hash: sha256:9af5554b44bc8b56df0ecf4501ae3ea18deeb0d8518e02ef6628f0ed61bf7309
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zquinlan/conCISE
  tools:
  - CANOPUS
  - ClassyFire
  - ConCISE
  - GNPS
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# grouped-bar-abundance-visualization

## Summary

Renders superclass-level ChemOnt relative abundances per organism (e.g., coral vs turfing-algal exudates from MSV000082083) as a grouped bar chart in matplotlib, after ConCISE consensus annotation, to expose compositional differences like organic acids enriched in corals and organoheterocyclic compounds enriched in turf algae.

## When to use

Use when you have a superclass-by-organism relative abundance table from ConCISE consensus annotations and need a grouped bar chart to visually compare chemical-class composition across exudate source organisms.

## When NOT to use

- The annotation output has not been aggregated to superclass-level relative abundances per organism (e.g., you only have raw per-feature annotations or class/subclass-level tables).
- The analysis target is a single organism or sample set with no grouping variable to compare across bar groups.
- You need statistical testing of class-level compositional differences rather than a descriptive visualization.

## Inputs

- ConCISE consensus annotation export file with ChemOnt superclass label per ion feature
- superclass-by-organism relative abundance matrix (superclass_abundance_table.csv)
- mapping of ion features to exudate source organism (Pocillopora verrucosa, Porites lobata, Dictyota ceylanica, turfing algae)

## Outputs

- figure3b_superclass_bars.png — grouped bar chart of superclass relative abundances by organism (matplotlib)
- superclass_abundance_table.csv — superclass-by-organism relative abundance matrix

## How to apply

After running the ConCISE CLI on the GNPS feature-based molecular networking task with the CANOPUS canopus_summary.tsv and Node_info.tsv inputs (default consensus thresholds: superclass 50%, class 70%, subclass 70%), parse the exported consensus annotation file and extract the ChemOnt superclass label assigned to each ion feature. Map each annotated feature to its exudate source organism (e.g., Pocillopora verrucosa, Porites lobata, Dictyota ceylanica, turfing algae) and compute superclass-level relative abundances per organism, writing the matrix to superclass_abundance_table.csv. Render the matrix as a grouped bar chart in matplotlib (figure3b_superclass_bars.png) with one bar group per superclass and one colored bar per organism. Validate that relative abundances sum to 1 within each organism column and that the directional patterns (organic acids higher in corals, organoheterocyclic compounds higher in turf algae) are visible in the rendered figure.

## Related tools

- **ConCISE** (Upstream tool producing the consensus annotation file whose superclass labels are aggregated and visualized) — https://github.com/Zquinlan/conCISE
- **Python** (Used to compute superclass relative abundances and render the grouped bar chart in matplotlib)

## Examples

```
python3 src/conciseCLI.py 16616afa8edd490ea7e50cc316a20222 exampleFiles/canopus_summary.tsv exampleFiles/Node_info.tsv 50 70 70 False
```

## Evaluation signals

- Relative abundances per organism sum to 1 within each organism column of superclass_abundance_table.csv.
- Organic acid superclass abundance is visually higher in coral (Pocillopora/Porites) exudate bars than in algal exudate bars.
- Organoheterocyclic compound superclass abundance is visually higher in turfing-algae exudate bars than in coral exudate bars.
- Bar groups correspond one-to-one to ChemOnt superclass labels extracted from the ConCISE consensus output; no superclass appears in the figure that is absent from the consensus file.
- QC thresholds trace to the ConCISE consensus defaults used upstream (superclass 50%, class 70%, subclass 70%).

## Limitations

- GNPS has stopped supplying ClassyFire ontology information for spectral library matches, so superclass labels may require the NPClassifier workaround or manual ClassyFire batch retrieval before abundance computation and visualization.
- The visualization reflects consensus-annotation coverage only; ConCISE annotated up to 76% of ion features, so unannotated features are excluded from the relative abundance denominators.
- The reproduced pattern is directional (relative enrichment), not an absolute quantification of exuded compounds.

## Evidence

- [results] At the subnetwork level, both species of coral produced relatively higher amounts of organic acids while the turfing algae released relatively high amounts of organoheterocyclic compounds in the coral reef exometabolome case study.: "At the subnetwork level, both species of coral produced relatively higher amounts of organic acids while the turfing algae released relatively high amounts of organoheterocyclic compounds"
- [readme] Superclass percent consensus (optional; default = 50): "Superclass percent consensus (optional; default = 50)"
- [readme] Currently GNPS has stopped supplying classyfire ontology information for spectral library matches.: "Currently GNPS has stopped supplying classyfire ontology information for spectral library matches. There are two workarounds: 1) use NPClassifier instead of ClassyFire by checking the box in the GUI"
- [abstract] The ConCISE framework expanded the proportion of reliable and consistent ion feature annotation up to 76%: "The ConCISE framework expanded the proportion of reliable and consistent ion feature annotation up to 76%"
