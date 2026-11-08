---
name: command-line-example-pipeline-execution
description: Use when the goal is to reproduce a published GraPhlAn visualization
  (e.g., the HMP aerobiosis cladogram, Fig. S1) by running the provided shell pipeline
  in an example folder such as hmp_aerobiosis rather than re-implementing the tool
  chain manually.
when_to_use_negative:
- Your input is raw sequencing reads — this pipeline starts from MetaPhlAn/LEfSe/HUMAnN
  outputs, not FASTQ files
- You need an interactive tree explorer (e.g., iTOL-style) rather than a publication-quality
  static circular figure
- Your data are not MetaPhlAn/LEfSe/HUMAnN outputs and cannot be expressed as a LEfSe-compatible
  input matrix
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3174
tools:
- name: export2graphlan
  role: Converts MetaPhlAn/LEfSe/HUMAnN outputs into the GraPhlAn tree.txt and annot.txt
    inputs within the example pipeline
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated tree (outtree.txt) into the circular phylogeny PNG images
    using matplotlib
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Graphics backend used by GraPhlAn to produce the final figure files
- name: hclust2
  role: Prerequisite library of the export2graphlan environment (SciPy dependency);
    companion heatmap tool in the same toolchain
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:486f4af7f5bada019d81ecb6fa234fdb9be5a934ba439ee8456562bad3bcd825
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# command-line-example-pipeline-execution

## Summary

Execute the bundled PIPELINE.sh script in an export2graphlan example sub-folder (e.g., hmp_aerobiosis) to chain export2graphlan, graphlan_annotate.py, and GraPhlAn/matplotlib end-to-end and reproduce the paper's figures from the authors' own example data.

## When to use

Use when the goal is to reproduce a published GraPhlAn visualization (e.g., the HMP aerobiosis cladogram, Fig. S1) by running the provided shell pipeline in an example folder such as hmp_aerobiosis rather than re-implementing the tool chain manually.

## When NOT to use

- Your input is raw sequencing reads — this pipeline starts from MetaPhlAn/LEfSe/HUMAnN outputs, not FASTQ files
- You need an interactive tree explorer (e.g., iTOL-style) rather than a publication-quality static circular figure
- Your data are not MetaPhlAn/LEfSe/HUMAnN outputs and cannot be expressed as a LEfSe-compatible input matrix

## Inputs

- LEfSe input abundance matrix (MetaPhlAn-style tab-separated taxonomic profile) in the example folder
- LEfSe biomarker result file (features with LDA scores)
- PIPELINE.sh bash script bundled in the export2graphlan examples folder

## Outputs

- tree.txt (GraPhlAn tree file)
- annot.txt (GraPhlAn annotation file)
- outtree.txt (annotated tree from graphlan_annotate.py)
- outimg.png (plain circular phylogeny rendered via matplotlib)
- outimg_annot.png (annotated cladogram with shading/labels)
- outimg_legend.png (biomarker legend)

## How to apply

Enter the example sub-folder (hmp_aerobiosis, hmp_metahit_metabolic, or hmp_metahit_mp2) of the export2graphlan repository and run the provided script via `./PIPELINE.sh`. The script first runs export2graphlan.py, which converts the LEfSe input matrix and LEfSe biomarker result into a GraPhlAn tree file (tree.txt) and annotation file (annot.txt), mapping LEfSe biomarker LDA scores to clade sizes (default clade size 10, biomarker clades 20–200) and class assignments to shaded background levels. It then applies graphlan_annotate.py to attach annot.txt to tree.txt, producing the annotated tree outtree.txt. Finally GraPhlAn (rendering via matplotlib) generates the circular phylogeny images outimg.png, outimg_annot.png, and outimg_legend.png. Rationale: the authors shipped these examples precisely so the whole conversion-and-rendering chain can be verified as a single command-driven step suitable for pipeline integration.

## Related tools

- **export2graphlan** (Converts MetaPhlAn/LEfSe/HUMAnN outputs into the GraPhlAn tree.txt and annot.txt inputs within the example pipeline) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated tree (outtree.txt) into the circular phylogeny PNG images using matplotlib) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Graphics backend used by GraPhlAn to produce the final figure files)
- **hclust2** (Prerequisite library of the export2graphlan environment (SciPy dependency); companion heatmap tool in the same toolchain) — https://github.com/SegataLab/hclust2

## Examples

```
cd export2graphlan/examples/hmp_aerobiosis && ./PIPELINE.sh
```

## Evaluation signals

- All six expected output files exist in the example folder after running PIPELINE.sh: annot.txt, outtree.txt, tree.txt, outimg.png, outimg_annot.png, outimg_legend.png
- outimg_annot.png visually matches the published aerobiosis cladogram (Fig. S1): circular phylogeny with shaded class-level backgrounds
- outtree.txt is non-empty and reflects annotations derived from annot.txt
- tree.txt and annot.txt map LEfSe biomarker LDA scores to clade node sizes within the 20–200 default range, with non-biomarker clades at default size 10
- The pipeline exits without pandas/SciPy/BIOM import errors (prerequisites per README)

## Limitations

- export2graphlan requires pandas ver. 0.13.1, BIOM ver. 2.0.1 (only for BIOM-format inputs), and SciPy, so environment version mismatches can break the pipeline
- No software version numbers are stated for GraPhlAn or export2graphlan in the article, and no changelog exists in the repository documentation, complicating exact reproducibility
- Mapping of LDA scores to clade sizes uses fixed defaults (min 20, max 200, threshold 20.0 abundance) that may need tuning for datasets other than the bundled examples

## Evidence

- [methods] Enter the hmp_aerobiosis example folder and run the provided script ($ ./PIPELINE.sh).: "The ``examples`` folder contains the following sub-folders: ``hmp_aerobiosis``, ``hmp_metahit_metabolic``, and ``hmp_metahit_mp2``."
- [discussion] export2graphlan.py converts the LEfSe input matrix and LEfSe biomarker result into a GraPhlAn tree file (tree.txt) and an annotation file (annot.txt): "The bash script provided executes export2graphlan for generating the annotation file and then invoking GraPhlAn for plotting the functional tree."
- [methods] outtree.txt is the annotated tree produced by graphlan_annotate.py: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [readme] export2graphlan requires pandas ver. 0.13.1 and BIOM ver. 2.0.1 as prerequisites: "**export2graphlan** requires the following additional library: * pandas ver. 0.13.1"
- [discussion] GraPhlAn uses the matplotlib library: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
