---
name: taxon-level-aggregation-summarization
description: Use when you have a genome-by-module presence/absence matrix (e.g., KEGG
  modules M00082, M00083, M00086, M00087, M00088, M00157, M00159) and need per-taxon
  counts or fractions of functional configurations to quantify mutual exclusivity
  or co-occurrence patterns.
when_to_use_negative:
- Input is already a per-taxon aggregated summary or a feature/abundance table rather
  than a per-genome presence/absence matrix — classification and grouping steps are
  unnecessary.
- The analysis requires continuous module abundance values (e.g., HUMAnN KO-group
  profiles) rather than binary presence/absence module calls; thresholding/normalization
  logic differs.
- You need interactive phylogeny browsing of large trees rather than compact publication
  figures — iTOL or Krona are alternatives the article positions against GraPhlAn.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: GraPhlAn
  role: Produces compact circular phylogenetic visualizations of the aggregated taxon-level
    functional metadata
  repo: https://bitbucket.org/nsegata/graphlan
- name: export2graphlan
  role: Converts tabular/LEfSe-style inputs into the GraPhlAn tree and annotation
    files after aggregation
  repo: https://github.com/SegataLab/export2graphlan
- name: pandas
  role: Loads the module presence/absence matrix and performs the groupby-taxon aggregation
    and category classification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:e4e78a283771e5e6b2c208b03eb0c107c091a5e6a0e3e7cc1fa287ab60cef439
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraPhlAn
  - export2graphlan
  - pandas
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# taxon-level-aggregation-summarization

## Summary

Aggregates per-genome KEGG module presence/absence data into per-taxon configuration categories (e.g., F-type-only, V/A-type-only, both, neither for ATPase modules M00157/M00159) with pandas, then flags exceptional clades and writes a summarized CSV suitable downstream for compact phylogenetic visualization with GraPhlAn.

## When to use

Use when you have a genome-by-module presence/absence matrix (e.g., KEGG modules M00082, M00083, M00086, M00087, M00088, M00157, M00159) and need per-taxon counts or fractions of functional configurations to quantify mutual exclusivity or co-occurrence patterns.

## When NOT to use

- Input is already a per-taxon aggregated summary or a feature/abundance table rather than a per-genome presence/absence matrix — classification and grouping steps are unnecessary.
- The analysis requires continuous module abundance values (e.g., HUMAnN KO-group profiles) rather than binary presence/absence module calls; thresholding/normalization logic differs.
- You need interactive phylogeny browsing of large trees rather than compact publication figures — iTOL or Krona are alternatives the article positions against GraPhlAn.

## Inputs

- Per-genome KEGG module presence/absence annotation matrix (rows = genomes, columns = modules including M00157 and M00159, derived from IMG/KEGG module set M00082, M00083, M00086, M00087, M00088, M00157, M00159)
- Genome-to-taxon mapping (taxonomic labels per genome row)

## Outputs

- atpase_configuration_summary.csv — per-taxon aggregation with the four configuration categories (F-type only, V/A-type only, both, neither), co-occurrence fractions, and an exceptional-clades flag column
- Taxon-level feature table suitable for conversion to GraPhlAn tree/annotation input via export2graphlan

## How to apply

Load the per-genome KEGG module presence/absence annotation matrix with pandas. For each genome row, classify the functional configuration into exactly one of four mutually exclusive categories — for the ATPase case: F-type only (M00157 present, M00159 absent), V/A-type only (M00159 present, M00157 absent), both, or neither. Group genomes by taxon and aggregate counts per category, computing the fraction of taxa showing co-occurrence of both systems so that the near-mutual exclusivity is quantified rather than reported only qualitatively. Flag exceptional taxa encoding both systems (e.g., Streptococcus and some Clostridia) in a dedicated column. Write the per-taxon aggregation to atpase_configuration_summary.csv, then optionally feed taxon-level features into GraPhlAn/export2graphlan for compact circular phylogeny rendering.

## Related tools

- **GraPhlAn** (Produces compact circular phylogenetic visualizations of the aggregated taxon-level functional metadata) — https://bitbucket.org/nsegata/graphlan
- **export2graphlan** (Converts tabular/LEfSe-style inputs into the GraPhlAn tree and annotation files after aggregation) — https://github.com/SegataLab/export2graphlan
- **pandas** (Loads the module presence/absence matrix and performs the groupby-taxon aggregation and category classification)

## Examples

```
df.assign(cfg=lambda d: d.apply(classify_atpase, axis=1)).groupby('taxon')['cfg'].value_counts().to_csv('atpase_configuration_summary.csv')
```

## Evaluation signals

- atpase_configuration_summary.csv exists and contains all four configuration categories (F-type only, V/A-type only, both, neither) as fields.
- Per-taxon category counts sum exactly to the total number of genomes in the input presence/absence matrix (mutual-exclusivity partition invariant).
- Both-systems taxa are restricted to exceptional clades consistent with the paper's qualitative report (e.g., Streptococcus and some Clostridia).
- Computed co-occurrence fractions are low, consistent with near-mutual exclusivity: V/A-type mainly in Archaea, F-type mostly in Bacteria.
- Downstream: export2graphlan produces both a tree and an annotation file that graphlan_annotate.py and graphlan.py can consume without errors.

## Limitations

- The near-mutual-exclusivity conclusion is qualitative from the source analysis; co-occurrence in exceptional clades (Streptococcus, Clostridia) means 'both' is a valid category and must not be discarded as noise.
- Module calls depend on the KEGG module definitions (M00082, M00083, M00086, M00087, M00088, M00157, M00159) and IMG genome annotation; errors in module calling propagate into the taxon aggregation.
- The README does not state software version numbers for export2graphlan or its dependencies (pandas, BIOM), so exact reproducibility of aggregation behavior across versions is not guaranteed.

## Evidence

- [discussion] From the KEGG database (Kanehisa & Goto, 2000; Kanehisa et al., 2014) we focused on the following modules: M00082, M00083, M00086, M00087, M00088, M00157, and M00159.: "From the KEGG database (Kanehisa & Goto, 2000; Kanehisa et al., 2014) we focused on the following modules: M00082, M00083, M00086, M00087, M00088, M00157, and M00159."
- [methods] Load the per-genome KEGG module presence/absence annotation matrix ... with pandas: [evidence span withheld — anchor status: not_found]
- [results] The F-type and V/A-type ATPases are almost mutually exclusive across annotated genomes: [evidence span withheld — anchor status: not_found]
- [abstract] We developed GraPhlAn (Graphical Phylogenetic Analysis), a computational tool that produces high-quality, compact visualizations of microbial genomes and metagenomes.: "We developed GraPhlAn (Graphical Phylogenetic Analysis), a computational tool that produces high-quality, compact visualizations of microbial genomes and metagenomes."
- [readme] export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn: "export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn."
