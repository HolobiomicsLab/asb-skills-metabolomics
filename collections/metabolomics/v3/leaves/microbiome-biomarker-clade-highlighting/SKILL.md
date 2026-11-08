---
name: microbiome-biomarker-clade-highlighting
description: Use when you need to render LEfSe biomarker results as a compact circular
  phylogenetic cladogram, e.g., highlighting taxonomic clades enriched in one cohort
  (HMP) versus another (MetaHIT) from MetaPhlAn taxonomic profiles.
when_to_use_negative:
- The goal is interactive exploration of a large phylogeny rather than a static publication
  figure (iTOL/Krona may fit better).
- Input is raw sequencing data or an unfiltered abundance table rather than processed
  MetaPhlAn/LEfSe files.
- The data are functional profiles not processed through LEfSe, so there are no biomarker
  clades to highlight (use the metabolic/HUMAnN workflow instead).
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: export2graphlan
  role: Converts the LEfSe input/output into the GraPhlAn tree.txt and annot.txt files,
    automatically flagging biomarker sub-trees for highlighting
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated tree as a high-quality circular cladogram via its matplotlib
    backend
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Graphics library used by GraPhlAn to produce the outimg.png render
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:82a4a2dd554d4d39bf4b2373ba21a7d38ebab663198606166137f3729f269512
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# microbiome-biomarker-clade-highlighting

## Summary

Reproduce the GraPhlAn circular taxonomic cladogram of HMP- vs. MetaHIT-enriched biomarker clades (Fig. 3) by converting MetaPhlAn/LEfSe outputs into a GraPhlAn tree and annotation file with export2graphlan, then rendering with GraPhlAn's matplotlib backend.

## When to use

Use when you need to render LEfSe biomarker results as a compact circular phylogenetic cladogram, e.g., highlighting taxonomic clades enriched in one cohort (HMP) versus another (MetaHIT) from MetaPhlAn taxonomic profiles.

## When NOT to use

- The goal is interactive exploration of a large phylogeny rather than a static publication figure (iTOL/Krona may fit better).
- Input is raw sequencing data or an unfiltered abundance table rather than processed MetaPhlAn/LEfSe files.
- The data are functional profiles not processed through LEfSe, so there are no biomarker clades to highlight (use the metabolic/HUMAnN workflow instead).

## Inputs

- LEfSe input file (MetaPhlAn taxonomic abundance table for healthy-individual HMP/MetaHIT samples)
- LEfSe output file listing significant biomarker clades

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (GraPhlAn annotation file)
- outtree.txt (annotated tree with annotation directives)
- outimg.png (circular cladogram PNG with highlighted biomarker clades)
- outimg_annot.png (annotation legend render)
- outimg_legend.png (biomarker legend render)

## How to apply

Start from the filtered LEfSe input (MetaPhlAn taxonomic profiles of healthy-individual samples) and the LEfSe output in the hmp_metahit example folder. Run export2graphlan.py with -i and -o pointing to the LEfSe input and output, and -t tree.txt and -a annot.txt as output tree and annotation files; the annotation file automatically highlights sub-trees that LEfSe identified as biomarkers. Annotate the generated tree with graphlan_annotate.py to produce outtree.txt containing the annotation directives, then run GraPhlAn on the annotated tree to render the circular cladogram as outimg.png with its matplotlib backend. Collect the additional renders (outimg_annot.png, outimg_legend.png) produced alongside the main plot for annotation-legend and biomarker-legend views.

## Related tools

- **export2graphlan** (Converts the LEfSe input/output into the GraPhlAn tree.txt and annot.txt files, automatically flagging biomarker sub-trees for highlighting) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated tree as a high-quality circular cladogram via its matplotlib backend) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Graphics library used by GraPhlAn to produce the outimg.png render)

## Examples

```
export2graphlan.py -i lefse_input.txt -o lefse_output.txt -t tree.txt -a annot.txt && graphlan_annotate.py tree.txt outtree.txt annot.txt && graphlan.py outtree.txt outimg.png
```

## Evaluation signals

- All six expected example outputs exist: annot.txt, tree.txt, outtree.txt, outimg.png, outimg_annot.png, outimg_legend.png.
- outimg.png is a valid PNG rendering of a circular tree.
- The rendered cladogram highlights clades enriched in HMP versus MetaHIT, consistent with the LEfSe biomarker list.
- outtree.txt contains the annotation directives attached by graphlan_annotate.py.
- The pipeline's expected outputs match those distributed in the GraPhlAn repository's examples folder.

## Limitations

- export2graphlan requires pandas 0.13.1, SciPy (needed by hclust2), and BIOM 2.0.1 only for BIOM-format inputs; missing dependencies will block execution.
- The finding depends on prior LEfSe analysis of the filtered healthy-individual dataset; without significant biomarkers, no sub-trees are highlighted.
- No software version numbers are stated for GraPhlAn or export2graphlan in the documentation, complicating exact reproduction.

## Evidence

- [readme] export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn.: "export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn."
- [methods] Run example PIPELINE.sh: "Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format"
- [methods] Annotated tree produced by graphlan_annotate.py: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [discussion] matplotlib backend: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
- [methods] Sum-up abundances from leaf to root on internal nodes: "--internal_levels     If specified sum-up from leaf to root the abundances values."
