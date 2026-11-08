---
name: metagenomic-taxonomic-cladogram-visualization
description: Use when you need to visualize taxonomic biomarkers contrasting metagenomic
  cohorts (e.g., LEfSe output on MetaPhlAn profiles) as a compact circular cladogram
  with highlighted, sized, and colored clades.
when_to_use_negative:
- Input is raw sequencing reads or an un-profiled abundance matrix — taxonomic profiling
  with MetaPhlAn2 and LEfSe biomarker analysis must be run first
- The goal is interactive exploration of large-scale phylogenies rather than a static
  publication figure — iTOL or Krona may be more appropriate
- Data are functional profiles (HUMAnN KEGG ortholog outputs) where a functional-tree
  rendering pipeline, not the taxonomic biomarker cladogram, is intended
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: export2graphlan
  role: Converts MetaPhlAn/LEfSe output into GraPhlAn tree.txt and annot.txt, highlighting
    biomarker sub-trees automatically
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated tree into the circular cladogram PNG via its matplotlib
    backend
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library used by GraPhlAn to produce the final image
- name: LEfSe
  role: Produces the biomarker input/output files on the filtered healthy-individual
    MetaPhlAn dataset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:f5434fa11d00245c88d9f3f035b31127addd61db5b027b67c46e329335d0adf0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - GraPhlAn
  - LEfSe
  - export2graphlan
  - matplotlib
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# metagenomic-taxonomic-cladogram-visualization

## Summary

Reproduce publication-quality circular taxonomic cladograms of microbiome biomarkers (e.g., HMP vs. healthy MetaHIT gut metagenomes, Fig. 3) by converting LEfSe biomarker results into GraPhlAn tree and annotation files with export2graphlan, then rendering with GraPhlAn's matplotlib backend.

## When to use

Use when you need to visualize taxonomic biomarkers contrasting metagenomic cohorts (e.g., LEfSe output on MetaPhlAn profiles) as a compact circular cladogram with highlighted, sized, and colored clades.

## When NOT to use

- Input is raw sequencing reads or an un-profiled abundance matrix — taxonomic profiling with MetaPhlAn2 and LEfSe biomarker analysis must be run first
- The goal is interactive exploration of large-scale phylogenies rather than a static publication figure — iTOL or Krona may be more appropriate
- Data are functional profiles (HUMAnN KEGG ortholog outputs) where a functional-tree rendering pipeline, not the taxonomic biomarker cladogram, is intended

## Inputs

- MetaPhlAn taxonomic abundance table (e.g., hmp_metahit.txt from the hmp_metahit example folder)
- LEfSe input file (class-labeled abundance table for biomarker analysis)
- LEfSe output file (biomarker results with effect sizes)

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (GraPhlAn annotation file)
- outtree.txt (annotated tree with directives from graphlan_annotate.py)
- outimg.png (circular cladogram PNG)
- outimg_annot.png (annotation legend render)
- outimg_legend.png (biomarker legend render)

## How to apply

Run LEfSe biomarker analysis on the filtered MetaPhlAn abundance table of healthy-individual samples, then execute export2graphlan.py with -i (LEfSe input) and -o (LEfSe output) to generate the tree file (-t tree.txt) and annotation file (-a annot.txt). Annotate the generated tree with graphlan_annotate.py to produce the annotated tree (outtree.txt) containing the annotation directives, sizing clades via LEfSe biomarker effect size within --min_clade_size 20 to --max_clade_size 200 and fonts between --min_font_size 8 and --max_font_size 12. Finally run GraPhlAn on the annotated tree to render the circular cladogram (outimg.png) with matplotlib, collecting the annotation-legend and biomarker-legend renders produced alongside. Key decision points: which taxonomic levels to annotate (--annotations), which clades get shaded backgrounds (--background_clades), and whether internal levels are summed leaf-to-root (--internal_levels).

## Related tools

- **export2graphlan** (Converts MetaPhlAn/LEfSe output into GraPhlAn tree.txt and annot.txt, highlighting biomarker sub-trees automatically) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated tree into the circular cladogram PNG via its matplotlib backend) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library used by GraPhlAn to produce the final image)
- **LEfSe** (Produces the biomarker input/output files on the filtered healthy-individual MetaPhlAn dataset)

## Examples

```
export2graphlan.py -i lefse_input.txt -o lefse_output.txt -t tree.txt -a annot.txt --internal_levels; graphlan_annotate.py tree.txt outtree.txt --annot annot.txt; graphlan.py outtree.txt outimg.png
```

## Evaluation signals

- All six example output files exist: annot.txt, tree.txt, outtree.txt, outimg.png, outimg_annot.png, outimg_legend.png
- outimg.png is a valid PNG depicting a circular tree with highlighted HMP- vs. MetaHIT-enriched clades
- tree.txt and annot.txt parse as valid GraPhlAn inputs and outtree.txt contains the annotation directives attached by graphlan_annotate.py
- Biomarker clades appear with sizes scaled between the min (20) and max (200) clade-size range and fonts between 8 and 12
- The rendered figure matches the pipeline's expected outputs from the hmp_metahit example folder

## Limitations

- export2graphlan requires specific dependency versions: pandas 0.13.1, BIOM 2.0.1 (only for BIOM-format inputs), and SciPy (required by hclust2)
- No software version numbers are stated for GraPhlAn, export2graphlan, or dependencies, complicating exact reproducibility
- No changelog is available in the repository documentation

## Evidence

- [methods] Workflow: export2graphlan + LEfSe + GraPhlAn on HMP/MetaHIT data: "Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format"
- [methods] Annotated tree output: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [methods] Sum internal abundance levels: "--internal_levels     If specified sum-up from leaf to root the abundances values."
- [readme] Clade size defaults from README: "Set a default size for clades that are not found as biomarkers by LEfSe. Default is 10"
- [readme] Tool purpose from README: "**export2graphlan** is a conversion software tool for producing both annotation and tree file for GraPhlAn."
- [discussion] matplotlib backend from discussion: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [abstract] Biomarker discovery example scope: "We present several examples including taxonomic and phylogenetic visualization of microbial communities, metabolic functions, and biomarker discovery"
