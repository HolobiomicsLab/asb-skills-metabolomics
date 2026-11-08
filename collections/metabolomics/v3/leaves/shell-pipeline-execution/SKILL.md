---
name: shell-pipeline-execution
description: Use when a task requires executing a multi-step command-line pipeline
  from a bash script — e.g., converting LEfSe/HUMAnN outputs to GraPhlAn inputs, annotating
  a tree, and rendering it as PNG.
when_to_use_negative:
- Input files are already in GraPhlAn tree/annotation format, so conversion via export2graphlan
  is unnecessary
- Input abundance tables are in BIOM format and pandas/biom-format prerequisites are
  unavailable
- The goal is interactive tree exploration rather than static publication-quality
  circular rendering (tools like iTOL or Krona target interactivity)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: export2graphlan
  role: Converts LEfSe input/output (from MetaPhlAn or HUMAnN profiles) into GraPhlAn
    tree.txt and annot.txt within the pipeline
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated functional tree image (backed by matplotlib) as the
    pipeline's final step
- name: matplotlib
  role: Graphics library backing GraPhlAn's rendering of the circular tree PNG
- name: hclust2
  role: Companion SegataLab tool installable via conda/pip that shares SciPy/pandas
    prerequisites with the export2graphlan environment
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:bf65a1299250e5a0fd5cda645b661a7775d92c6f7161a447ec7789aa8f1f0684
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - GraPhlAn
  - export2graphlan
  - hclust2
  - matplotlib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# shell-pipeline-execution

## Summary

Chains command-line tools (export2graphlan.py, graphlan_annotate.py, GraPhlAn/matplotlib) via a bash PIPELINE.sh script to convert HUMAnN/LEfSe functional profiles into annotated GraPhlAn tree files and a rendered circular tree image.

## When to use

Use when a task requires executing a multi-step command-line pipeline from a bash script — e.g., converting LEfSe/HUMAnN outputs to GraPhlAn inputs, annotating a tree, and rendering it as PNG.

## When NOT to use

- Input files are already in GraPhlAn tree/annotation format, so conversion via export2graphlan is unnecessary
- Input abundance tables are in BIOM format and pandas/biom-format prerequisites are unavailable
- The goal is interactive tree exploration rather than static publication-quality circular rendering (tools like iTOL or Krona target interactivity)

## Inputs

- lefse_input: HUMAnN-derived functional profile matrix (KEGG pathways x samples) formatted for LEfSe
- lefse_output: LEfSe biomarker result file
- PIPELINE.sh: bash script orchestrating export2graphlan, graphlan_annotate.py, and GraPhlAn

## Outputs

- tree.txt: GraPhlAn tree file
- annot.txt: GraPhlAn annotation file
- outtree.txt: annotated tree produced by graphlan_annotate.py
- outimg.png: rendered circular functional-ontology tree
- outimg_annot.png and outimg_legend.png: annotated rendering and legend images

## How to apply

Enter the example folder (e.g., hmp_metahit_metabolic) containing the HUMAnN-derived LEfSe input matrix (lefse_input) and the LEfSe result file (lefse_output). Run ./PIPELINE.sh, which invokes export2graphlan.py with -i lefse_input and -o lefse_output to emit tree.txt and annot.txt; use --internal_levels to sum abundances from leaf to root on internal nodes when needed. Annotate the exported tree with graphlan_annotate.py to produce outtree.txt, then invoke GraPhlAn (backed by matplotlib) to render the annotated functional tree. The rationale is that export2graphlan's two outputs are designed to feed graphlan_annotate.py and then graphlan.py, keeping the conversion, annotation, and rendering steps reproducible in a single command-driven script suitable for publication pipelines.

## Related tools

- **export2graphlan** (Converts LEfSe input/output (from MetaPhlAn or HUMAnN profiles) into GraPhlAn tree.txt and annot.txt within the pipeline) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated functional tree image (backed by matplotlib) as the pipeline's final step)
- **matplotlib** (Graphics library backing GraPhlAn's rendering of the circular tree PNG)
- **hclust2** (Companion SegataLab tool installable via conda/pip that shares SciPy/pandas prerequisites with the export2graphlan environment) — https://github.com/SegataLab/hclust2

## Examples

```
./PIPELINE.sh
```

## Evaluation signals

- All six expected files exist: annot.txt, outtree.txt, tree.txt, outimg.png, outimg_annot.png, outimg_legend.png
- outimg.png is a non-empty rendered circular tree of the functional ontology
- export2graphlan exits cleanly on lefse_input/lefse_output, producing both tree.txt and annot.txt
- outtree.txt differs from tree.txt by containing annotations attached by graphlan_annotate.py
- Rendered tree reproduces the Fig. 4 pattern: Glycolysis and Citrate cycle enriched in HMP, Sulfur Metabolism and Vitamin B6 Metabolism enriched in MetaHIT

## Limitations

- export2graphlan requires pandas ver. 0.13.1, biom-format ver. 2.0.1 (only for BIOM inputs), and SciPy; missing dependencies break the pipeline
- The pipeline is command-driven and batch-oriented; it does not provide the interactive exploration offered by iTOL or Krona
- No changelog found in the repository documentation and no software version numbers stated for GraPhlAn, export2graphlan, or dependencies in the article, complicating exact reproducibility

## Evidence

- [discussion] The bash script provided executes export2graphlan for generating the annotation file and then invoking GraPhlAn for plotting the functional tree.: "The bash script provided executes export2graphlan for generating the annotation file and then invoking GraPhlAn for plotting the functional tree."
- [methods] annotated tree produced by graphlan_annotate.py: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [readme] run graphlan_annotate.py to attach annotations, then graphlan.py for the output image: "run ``graphlan_annotate.py``, in order to attach to the tree the derived annotations"
- [methods] Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format: "Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format"
