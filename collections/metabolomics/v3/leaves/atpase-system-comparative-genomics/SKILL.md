---
name: atpase-system-comparative-genomics
description: Use when you have a per-genome KEGG module presence/absence matrix derived
  from IMG/KEGG modules (M00082, M00083, M00086, M00087, M00088, M00157, M00159) and
  need to determine which taxa encode F-type vs V/A-type ATPases, or both.
when_to_use_negative:
- Input is raw sequence or ortholog calls rather than an already-derived KEGG module
  presence/absence matrix — module reconstruction must happen first.
- The goal is interactive exploration of a large tree with overlaid metadata rather
  than a compact, reproducible publication figure; the paper positions iTOL or Krona
  as alternatives for interactive analysis.
- The ATPase system of interest is not covered by KEGG modules M00157/M00159 (e.g.
  other ion-pumping ATPase families).
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0157
tools:
- name: pandas
  role: Loads the per-genome KEGG module presence/absence matrix and performs the
    per-taxon aggregation of ATPase configuration categories
  repo: http://pandas.pydata.org/index.html
- name: export2graphlan
  role: Converts the analysis output into GraPhlAn tree and annotation files; depends
    on pandas 0.13.1 and optionally BIOM 2.0.1 and SciPy (via hclust2)
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the compact circular phylogeny annotated with per-taxon ATPase-configuration
    metadata; built on matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: hclust2
  role: Optional companion for heat-map visualization of the module presence/absence
    or aggregated matrix
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:280247054d446795f97950b65b01b392598fb68a33712c64417c4c8189bd97c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraPhlAn
  - export2graphlan
  - hclust2
  - pandas
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# atpase-system-comparative-genomics

## Summary

Classifies per-genome presence/absence of KEGG ATP synthase modules M00157 (F-type) and M00159 (V/A-type) across annotated microbial genomes, quantifies their near-mutual exclusivity by taxon, flags exceptional dual-system clades (Streptococcus, some Clostridia), and renders the result as a compact GraPhlAn circular phylogeny with ATPase-configuration metadata.

## When to use

Use when you have a per-genome KEGG module presence/absence matrix derived from IMG/KEGG modules (M00082, M00083, M00086, M00087, M00088, M00157, M00159) and need to determine which taxa encode F-type vs V/A-type ATPases, or both.

## When NOT to use

- Input is raw sequence or ortholog calls rather than an already-derived KEGG module presence/absence matrix — module reconstruction must happen first.
- The goal is interactive exploration of a large tree with overlaid metadata rather than a compact, reproducible publication figure; the paper positions iTOL or Krona as alternatives for interactive analysis.
- The ATPase system of interest is not covered by KEGG modules M00157/M00159 (e.g. other ion-pumping ATPase families).

## Inputs

- Per-genome KEGG module presence/absence annotation matrix (tabular, e.g. TSV/CSV, rows = genomes, columns include M00157 and M00159; module set derived from M00082, M00083, M00086, M00087, M00088, M00157, M00159)
- Per-genome taxonomic assignments (for grouping by taxon)
- Phylogenetic tree of the annotated genomes (for GraPhlAn rendering)

## Outputs

- atpase_configuration_summary.csv: per-taxon aggregation with the four configuration categories (F-type only, V/A-type only, both, neither), co-occurrence fraction, and a dual-systems exception flag
- GraPhlAn circular phylogeny image annotated with ATPase-configuration metadata (tree and annotation files produced via graphlan_annotate.py and export2graphlan)

## How to apply

Load the per-genome KEGG module presence/absence matrix with pandas and, for each genome row, classify the ATPase configuration into exactly one of four categories: F-type only (M00157 present, M00159 absent), V/A-type only (M00159 present, M00157 absent), both, or neither. Group genomes by taxon, aggregate counts per configuration category, and compute the fraction of taxa with co-occurrence of both systems to quantify the near-mutual exclusivity (V/A-type mainly in Archaea, F-type mostly in Bacteria) that the paper reports qualitatively. Add a dedicated flag column for exceptional taxa encoding both systems, restricted to clades such as Streptococcus and some Clostridia. Write the per-taxon aggregation to atpase_configuration_summary.csv and validate that per-taxon counts sum to the total number of genomes in the input matrix. For publication-quality visualization, map the per-taxon ATPase configuration onto the phylogeny and render it with GraPhlAn (optionally generating tree/annotation files via export2graphlan, which requires pandas 0.13.1, optionally BIOM 2.0.1, and SciPy).

## Related tools

- **pandas** (Loads the per-genome KEGG module presence/absence matrix and performs the per-taxon aggregation of ATPase configuration categories) — http://pandas.pydata.org/index.html
- **export2graphlan** (Converts the analysis output into GraPhlAn tree and annotation files; depends on pandas 0.13.1 and optionally BIOM 2.0.1 and SciPy (via hclust2)) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the compact circular phylogeny annotated with per-taxon ATPase-configuration metadata; built on matplotlib) — https://bitbucket.org/nsegata/graphlan
- **hclust2** (Optional companion for heat-map visualization of the module presence/absence or aggregated matrix) — https://github.com/SegataLab/hclust2

## Examples

```
import pandas as pd; m = pd.read_csv('module_matrix.tsv', sep='\t', index_col=0); cfg = ((m['M00157']>0).astype(int)*2 + (m['M00159']>0).astype(int)).map({2:'F-type only',1:'V/A-type only',3:'both',0:'neither'}); cfg.to_frame('atpase_config').to_csv('atpase_configuration_summary.csv')
```

## Evaluation signals

- atpase_configuration_summary.csv exists and contains all four configuration categories (F-type only, V/A-type only, both, neither) as fields.
- Per-taxon counts across the four categories sum to the total number of genome rows in the input presence/absence matrix (each genome classified into exactly one category).
- Taxa flagged as encoding both systems are restricted to the exceptional clades consistent with the paper's qualitative report (Streptococcus and some Clostridia).
- The computed co-occurrence fraction is low, reproducing the near-mutual exclusivity pattern: V/A-type mainly in Archaea and F-type mostly in Bacteria.
- The GraPhlAn figure reproduces the annotated tree: graphlan_annotate.py produces the annotated outtree.txt and graphlan.py renders it without annotation errors.

## Limitations

- The mutual-exclusivity conclusion depends on the quality and completeness of the underlying IMG/KEGG module annotations; partially reconstructed ATPase modules can misclassify genomes as 'neither'.
- Exceptional dual-system clades may be missed or mislabeled if the taxonomic grouping is coarser than genus level.
- No software version numbers are stated for GraPhlAn, export2graphlan, or dependencies in the article; only export2graphlan ver. 0.2.1 (27 October 2018) and pandas 0.13.1 / BIOM 2.0.1 prerequisites are documented in the README.
- The near-mutual exclusivity is assessed qualitatively; the co-occurrence fraction is a descriptive statistic, not a significance test.

## Evidence

- [discussion] KEGG modules used for the ATPase analysis: "From the KEGG database (Kanehisa & Goto, 2000; Kanehisa et al., 2014) we focused on the following modules: M00082, M00083, M00086, M00087, M00088, M00157, and M00159."
- [abstract] GraPhlAn purpose and scale: "We developed GraPhlAn (Graphical Phylogenetic Analysis), a computational tool that produces high-quality, compact visualizations of microbial genomes and metagenomes."
- [readme] export2graphlan output and downstream commands: "The two output file of **export2graphlan** should then be used to run ``graphlan_annotate.py``, in order to attach to the tree the derived annotations, and finally, by executing ``graphlan.py`` the"
- [readme] export2graphlan prerequisites: "* pandas ver. 0.13.1 ([pandas](http://pandas.pydata.org/index.html))
* BIOM ver. 2.0.1 ([biom-format](http://biom-format.org), only if you have input files in BIOM format)"
- [discussion] GraPhlAn rendering engine: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
