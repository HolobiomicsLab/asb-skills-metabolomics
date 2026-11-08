---
name: graphlan-legend-and-annotation-configuration
description: Use when converting LEfSe/MetaPhlAn biomarker results into a GraPhlAn
  tree plus annotation file and you need controlled highlight levels, background clades/colors,
  font and clade sizing, and legend rendering.
when_to_use_negative:
- Input is already a GraPhlAn tree/annotation pair and only rendering is needed —
  no export2graphlan conversion step required.
- Data are not MetaPhlAn, LEfSe, or HUMAnN outputs (export2graphlan explicitly converts
  only these formats).
- The goal is interactive exploration of large phylogenies rather than a compact publication
  figure — iTOL or Krona target interactive analyses.
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: export2graphlan
  role: Converts MetaPhlAn/LEfSe output into the GraPhlAn tree.txt and annot.txt,
    setting which levels and clades are annotated, shaded, and sized
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Consumes the annotated tree (outtree.txt) and renders the circular cladogram
    and its annotation/biomarker legend images via matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Rendering backend GraPhlAn uses to produce the PNG figure and legend files
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:ea251a6d9f33659498358bb12464e8641a403b7fd3c627c81f722220f214091a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - GraPhlAn
  - export2graphlan
  - matplotlib
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# graphlan-legend-and-annotation-configuration

## Summary

Configure GraPhlAn annotation directives and legends so a circular cladogram highlights biomarker clades with shaded backgrounds, sized clades, and annotation/external legends, as done for the HMP vs. MetaHIT Fig. 3 reproduction.

## When to use

Use when converting LEfSe/MetaPhlAn biomarker results into a GraPhlAn tree plus annotation file and you need controlled highlight levels, background clades/colors, font and clade sizing, and legend rendering.

## When NOT to use

- Input is already a GraPhlAn tree/annotation pair and only rendering is needed — no export2graphlan conversion step required.
- Data are not MetaPhlAn, LEfSe, or HUMAnN outputs (export2graphlan explicitly converts only these formats).
- The goal is interactive exploration of large phylogenies rather than a compact publication figure — iTOL or Krona target interactive analyses.

## Inputs

- LEfSe input file (MetaPhlAn taxonomic profile matrix, e.g. hmp_metahit.txt)
- LEfSe output file (biomarker result from the filtered healthy-individual dataset)
- export2graphlan options: annotation levels, background clades/colors, clade and font sizes, abundance threshold

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (GraPhlAn annotation file)
- outtree.txt (annotated tree produced by graphlan_annotate.py)
- outimg.png (rendered circular cladogram with highlighted clades)
- outimg_annot.png (annotation-legend render)
- outimg_legend.png (biomarker-legend render)

## How to apply

Run export2graphlan.py on the LEfSe input/output (-i/-o) to emit tree.txt and annot.txt, tuning --background_levels/--background_clades and --background_colors to shade the clades enriched in HMP vs. healthy MetaHIT samples, and using --def_clade_size/--min_clade_size/--max_clade_size and font-size options to scale biomarker clades and labels. Attach the directives with graphlan_annotate.py to produce outtree.txt (the annotated tree), then render with GraPhlAn's matplotlib backend to obtain outimg.png plus the companion outimg_annot.png and outimg_legend.png legend renders. The abundance_threshold (default 20.0) and --most_abundant/--least_biomarkers options decide which clades get annotated, and --internal_levels sums leaf-to-root abundances on internal nodes so higher ranks can be highlighted.

## Related tools

- **export2graphlan** (Converts MetaPhlAn/LEfSe output into the GraPhlAn tree.txt and annot.txt, setting which levels and clades are annotated, shaded, and sized) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Consumes the annotated tree (outtree.txt) and renders the circular cladogram and its annotation/biomarker legend images via matplotlib) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Rendering backend GraPhlAn uses to produce the PNG figure and legend files)

## Examples

```
export2graphlan.py -i lefse_input.txt -o lefse_output.txt -t tree.txt -a annot.txt --annotations 1,2,3 --background_levels 2,3; graphlan_annotate.py annot.txt tree.txt --annotations outtree.txt; graphlan.py outtree.txt outimg.png
```

## Evaluation signals

- All six example outputs exist: annot.txt, tree.txt, outtree.txt, outimg.png, outimg_annot.png, outimg_legend.png.
- outimg.png is a valid PNG depicting a circular tree with highlighted HMP- vs. MetaHIT-enriched clades.
- outtree.txt contains the annotation directives attached by graphlan_annotate.py on top of tree.txt.
- Legend renders (outimg_annot.png, outimg_legend.png) are produced alongside the main plot, confirming annotation-legend and biomarker-legend configuration.
- Clade sizes fall between --min_clade_size (20) and --max_clade_size (200) defaults for biomarkers, with default 10 for non-biomarker clades.

## Limitations

- No software version numbers are stated for GraPhlAn, export2graphlan, or dependencies (matplotlib, pandas, BIOM), so exact reproducibility of rendering may vary.
- export2graphlan requires pandas 0.13.1, BIOM 2.0.1 (only for BIOM-format inputs), and SciPy (required by hclust2).
- Annotated clades are limited by the default abundance_threshold of 20.0, so low-abundance biomarkers may be silently omitted.

## Evidence

- [readme] graphlan_annotate and legend pipeline: "The two output file of **export2graphlan** should then be used to run ``graphlan_annotate.py``, in order to attach to the tree the derived annotations"
- [readme] annotation directives control: "--background_clades "Bacteria.Actinobacteria, Bacteria.Bacteroidetes.Bacteroidia"
- [methods] annotated tree output: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [discussion] matplotlib rendering: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [discussion] input file provenance: "The two input files represent the merge result of the MetaPhlAn analysis (hmp_metahit.txt) and the LEfSe result on the first file"
