---
name: taxonomic-tree-generation-for-graphlan
description: Use when you have a taxonomy-structured abundance or biomarker table
  (MetaPhlAn/LEfSe/HUMAnN output, BIOM v1/v2, e.g. 16S vaginal microbiota) and need
  a GraPhlAn tree + annotation pair for circular rendering.
when_to_use_negative:
- Input is an unrooted phylogenetic tree or non-taxonomic hierarchy with no abundance/annotation
  table — export2graphlan expects taxonomy-labeled abundance/biomarker rows
- You need interactive exploration of large-scale phylogenies rather than a static
  publication figure — tools like iTOL target interactive analyses
- No biom-format library installed and input is BIOM — the README states BIOM ver.
  2.0.1 is required only if input files are in BIOM format
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: export2graphlan
  role: Converts MetaPhlAn/LEfSe/HUMAnN or BIOM input into the GraPhlAn tree.txt and
    annot.txt files; --internal_levels sums abundances leaf-to-root
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated tree (outtree.txt) into the circular taxonomy image
    and legend images using matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library GraPhlAn uses to draw the circular phylogeny
- name: hclust2
  role: Companion tool for heat-map visualization of the same MetaPhlAn abundance
    profiles; listed as a prerequisite (SciPy) for export2graphlan
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:e0fa0bab6e961d0cfd9804729d6145fee87272f4eab016d68849b163963b59cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - GraPhlAn
  - export2graphlan
  - hclust2
  - matplotlib
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# taxonomic-tree-generation-for-graphlan

## Summary

Converts taxonomic abundance tables (MetaPhlAn, LEfSe, HUMAnN outputs, or BIOM v1/v2 feature matrices such as 16S rRNA vaginal microbiota profiles) into a GraPhlAn tree file and annotation file, enabling circular, publication-quality phylogenetic visualizations via export2graphlan, graphlan_annotate.py, and GraPhlAn/matplotlib.

## When to use

Use when you have a taxonomy-structured abundance or biomarker table (MetaPhlAn/LEfSe/HUMAnN output, BIOM v1/v2, e.g. 16S vaginal microbiota) and need a GraPhlAn tree + annotation pair for circular rendering.

## When NOT to use

- Input is an unrooted phylogenetic tree or non-taxonomic hierarchy with no abundance/annotation table — export2graphlan expects taxonomy-labeled abundance/biomarker rows
- You need interactive exploration of large-scale phylogenies rather than a static publication figure — tools like iTOL target interactive analyses
- No biom-format library installed and input is BIOM — the README states BIOM ver. 2.0.1 is required only if input files are in BIOM format

## Inputs

- MetaPhlAn/LEfSe/HUMAnN output table (e.g. hmp_metahit.txt) or LEfSe biomarker result file
- BIOM-format abundance matrix (BIOM v1 or v2), e.g. from QIIME or mothur 16S rRNA studies such as vaginal microbiota profiles

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (GraPhlAn annotation file)
- outtree.txt (annotated tree from graphlan_annotate.py)
- outimg.png (circular taxonomy tree rendered by GraPhlAn)
- outimg_annot.png (annotation legend image)
- outimg_legend.png (highlighted-biomarker legend image)

## How to apply

Run export2graphlan.py on the abundance/biomarker input (BIOM v1/v2 is accepted directly, enabling conversion of QIIME- and mothur-derived 16S profiles), passing --internal_levels so abundances are summed from leaf to root onto internal nodes; the script emits tree.txt and annot.txt. Annotate the tree with graphlan_annotate.py to attach the derived annotations, producing outtree.txt. Render with GraPhlAn (which relies on matplotlib) to obtain the circular taxonomy image outimg.png plus the annotation-legend (outimg_annot.png) and biomarker-legend (outimg_legend.png) images. export2graphlan depends on pandas (>=0.13.1), BIOM (2.0.1, only for BIOM inputs), and SciPy; it is installable via Bioconda or pip. Size and emphasis of clades can be tuned with --def_clade_size, --min_clade_size/--max_clade_size (defaults 10/20/200) and font-size options; --abundance_threshold (default 20.0) gates which clades get annotated.

## Related tools

- **export2graphlan** (Converts MetaPhlAn/LEfSe/HUMAnN or BIOM input into the GraPhlAn tree.txt and annot.txt files; --internal_levels sums abundances leaf-to-root) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated tree (outtree.txt) into the circular taxonomy image and legend images using matplotlib) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library GraPhlAn uses to draw the circular phylogeny)
- **hclust2** (Companion tool for heat-map visualization of the same MetaPhlAn abundance profiles; listed as a prerequisite (SciPy) for export2graphlan) — https://github.com/SegataLab/hclust2

## Examples

```
export2graphlan.py -i vaginal_biom.biom --internal_levels -t tree.txt -a annot.txt && graphlan_annotate.py annot.txt tree.txt --annot outtree.txt && graphlan.py outtree.txt outimg.png
```

## Evaluation signals

- The run produces the six expected files: annot.txt, outimg.png, outimg_annot.png, outimg_legend.png, outtree.txt, tree.txt
- outimg.png displays a circular tree consistent with the reference figure (e.g. Fig. S3 temporal-dynamics vaginal microbiota representation)
- With --internal_levels, internal-node abundance values equal the summed leaf-to-root abundances of their descendants
- annot.txt only annotates clades whose abundance exceeds --abundance_threshold (default 20.0) and biomarker clades within --min_clade_size/--max_clade_size (20/200)
- outtree.txt is a syntactically valid annotated tree that graphlan.py can render without errors

## Limitations

- No software version numbers are stated in the article for GraPhlAn, export2graphlan, or dependencies (matplotlib, pandas, BIOM); README specifies pandas 0.13.1 and BIOM 2.0.1, so newer versions may need testing
- BIOM support requires the biom-format library installed; without it BIOM inputs fail
- No changelog is available in the repository documentation, complicating version tracking
- OTU-level leaves can be discarded via --discard_otus; very deep taxonomies rely on --most_abundant / --least_biomarkers to keep the plot compact

## Evidence

- [results] BIOM input support: [evidence span withheld — anchor status: not_found]
- [methods] internal levels flag: "--internal_levels     If specified sum-up from leaf to root the abundances values."
- [methods] annotate step: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [discussion] matplotlib dependency: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [readme] prerequisites: "BIOM ver. 2.0.1 ([biom-format](http://biom-format.org), only if you have input files in BIOM format)"
- [readme] tool purpose: "**export2graphlan** is a conversion software tool for producing both annotation and tree file for GraPhlAn."
- [readme] default thresholds: "--abundance_threshold ABUNDANCE_THRESHOLD
                        Set the minimun abundace value for a clade to be
                        annotated. Default is 20.0"
