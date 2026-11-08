---
name: chemotaxonomic-distribution-analysis
description: Use when annotated molecular network nodes (with CF_superclass/class/subclass
  labels) must be summarized per taxon or clade, e.g. contrasting metabolite classes
  between plant clades or bacterial groups.
when_to_use_negative:
- Network nodes lack ClassyFire chemical class annotations (CF_ columns absent), so
  there is nothing to aggregate per taxon.
- No taxonomic grouping metadata exists for the molecular families, preventing clade-level
  contrast.
- The goal is per-node annotation rather than group-level class distribution comparison
  — direct node-level CF columns in Cytoscape suffice.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3292
tools:
- name: pyMolNetEnhancer
  role: generates the class- and motif-annotated GraphML that the distribution aggregation
    consumes; make_classyfire_graphml merges motif-mapped network 'MG' with class
    dataframe 'final'
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: ClassyFire
  role: provides the superclass/class/subclass taxonomy (CF_* columns with scores)
    that is aggregated per clade
- name: GNPS
  role: supplies the molecular network and library hits whose node-level classes are
    summarized chemotaxonomically
- name: MS2LDA
  role: supplies Mass2Motif substructure assignments (e.g. motif 130, 86, 149) that
    serve as validation anchors in the distribution output
- name: Network Annotation Propagation
  role: one of the in silico annotation inputs whose SMILES feed ClassyFire classification
    for the aggregated table
- name: DEREPLICATOR
  role: complementary in silico annotation source whose SMILES feed ClassyFire classification
    for the aggregated table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:a55002f26d5ce12f7ecbc2c47aaeb95a1cb3ae9bb12fb717a4a7479b1ec65c48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - ClassyFire
  - DEREPLICATOR
  - GNPS
  - MS2LDA
  - MolNetEnhancer
  - Network Annotation Propagation
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# chemotaxonomic-distribution-analysis

## Summary

Aggregates MolNetEnhancer-annotated chemical class assignments (ClassyFire CF_subclass etc.) per taxonomic grouping or network component, normalizing counts to build class-distribution tables that contrast metabolite profiles between taxa.

## When to use

Use when annotated molecular network nodes (with CF_superclass/class/subclass labels) must be summarized per taxon or clade, e.g. contrasting metabolite classes between plant clades or bacterial groups.

## When NOT to use

- Network nodes lack ClassyFire chemical class annotations (CF_ columns absent), so there is nothing to aggregate per taxon.
- No taxonomic grouping metadata exists for the molecular families, preventing clade-level contrast.
- The goal is per-node annotation rather than group-level class distribution comparison — direct node-level CF columns in Cytoscape suffice.

## Inputs

- feature-based GNPS molecular networking output (e.g. MASSIVE accession MSV000081805)
- MS2LDA experiment motif summary file (e.g. ms2lda.org experiment 566, downloaded manually if server timeout)
- NAP and DEREPLICATOR in silico annotation TSVs with numeric 'Scan' column matching GNPS node IDs and a 'SMILES' column
- GNPS library hit table
- taxonomic metadata assigning network nodes/families to clades
- annotated .graphml and .txt tables from pyMolNetEnhancer

## Outputs

- normalized chemical-class distribution table per taxonomic group (e.g. CF_subclass × clade)
- combined chemical-class + Mass2Motif GraphML (make_classyfire_graphml output)
- Cytoscape-loadable .graphml with CF_subclass as node fill-color attribute

## How to apply

Start from the annotated .graphml/.txt produced by pyMolNetEnhancer after ClassyFire taxonomy (CF_superclass, CF_class, CF_subclass with *_score columns) and optionally MS2LDA Mass2Motif mapping (prob 0.01, overlap 0.3, top 5 defaults) have been joined to the GNPS network via make_classyfire_graphml. Group nodes by the taxonomic attribute of interest (e.g. Rhamnoid vs Ziziphoid clade labels) or by network component index, then aggregate CF_subclass assignments per group. Normalize the aggregated counts so class abundance is comparable across groups, yielding a distribution table. Validate by checking the output reproduces known chemical-class contrasts (e.g. flavonoids/carbohydrates/anthraquinones enriched in Rhamnoid, triterpenoids in Ziziphoid) and that diagnostic motifs such as kaempferol-related Mass2Motif 130 (m/z 284/255/227) and quercetin-related Motifs 86/149 appear in the mapped output. Load the final GraphML into Cytoscape and confirm 'CF_subclass' is available as a node fill-color column.

## Related tools

- **pyMolNetEnhancer** (generates the class- and motif-annotated GraphML that the distribution aggregation consumes; make_classyfire_graphml merges motif-mapped network 'MG' with class dataframe 'final') — https://github.com/madeleineernst/pyMolNetEnhancer
- **ClassyFire** (provides the superclass/class/subclass taxonomy (CF_* columns with scores) that is aggregated per clade)
- **GNPS** (supplies the molecular network and library hits whose node-level classes are summarized chemotaxonomically)
- **MS2LDA** (supplies Mass2Motif substructure assignments (e.g. motif 130, 86, 149) that serve as validation anchors in the distribution output)
- **Network Annotation Propagation** (one of the in silico annotation inputs whose SMILES feed ClassyFire classification for the aggregated table)
- **DEREPLICATOR** (complementary in silico annotation source whose SMILES feed ClassyFire classification for the aggregated table)

## Examples

```
pip install pyMolNetEnhancer
```

## Evaluation signals

- Aggregate CF_subclass counts per group and verify normalization makes distributions comparable across clades.
- Confirm reported chemical contrast reproduces: Rhamnoid clades enriched in flavonoids, carbohydrates and anthraquinones; Ziziphoid clades enriched in triterpenoids and triterpenoid glycosides.
- Verify diagnostic Mass2Motifs (kaempferol-related Motif 130 m/z 284/255/227; quercetin-related Motifs 86/149) are present in the motif-mapped output.
- Check the final .graphml loads into Cytoscape with 'CF_subclass' available as a node fill-color column.
- Confirm each annotation input TSV has a numeric 'Scan' column that matches GNPS node identifiers and a 'SMILES' column.

## Limitations

- ms2lda.org may return a server connection timeout when exporting large summary files; the motif summary must then be downloaded manually.
- ClassyFire scores (CF_*_score columns) should be inspected — low-confidence class assignments will degrade the distribution table.
- Default motif-mapping thresholds (prob 0.01, overlap 0.3, top 5) filter which Mass2Motif–node relations enter the combined graphml; adjusting them changes the distribution analysis inputs.
- Differential outputs from PyMolNetEnhancer vs DEREPLICATOR are complementary, not redundant — using only one may under-annotate classes available for aggregation.

## Evidence

- [results] The taxonomic clade Rhamnoid exhibits more diversified flavonoids, carbohydrates, and anthraquinones, while the Ziziphoid clade produces various triterpenoids and triterpenoid glycosides.: [evidence span withheld — anchor status: not_found]
- [abstract] MolNetEnhancer enables chemical annotation, visualization, and discovery of subtle substructural diversity: "MolNetEnhancer enables the chemical annotation, visualization, and discovery of the subtle substructural diversity within"
- [methods] Map combined chemical class and MS2LDA substructural information to mass spectral molecular networks: "Map chemical class and MS2LDA substructural information to mass spectral molecular networks"
- [readme] Make combined chemical class and Mass2Motif GraphML; the motif-mapped network 'MG' and the class dataframe 'final' are merged with defaults prob=0.01, overlap=0.3, top=5: "<tt>prob</tt>: minimal probability score for a Mass2Motif to be included. Default is 0.01. <br>
  <tt>overlap</tt>: minimal overlap score for a Mass2Motif to be included. Default is 0.3. <br>"
- [other] Verify the GraphML loads into Cytoscape with 'CF_subclass' available as a node fill-color column: "To color nodes by the most shared Mass2Motifs per molecular family (network component index) select 'Image/Chart' in the 'Node' tab to the left and select Mass2Motifs shown in 'TopSharedMotifs' in"
