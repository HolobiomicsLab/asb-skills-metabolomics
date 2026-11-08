---
name: command-line-bioinformatics-pipelining
description: Use when a pipeline must convert metagenomic profiler outputs (MetaPhlAn
  v1/v2, HUMAnN, LEfSe results) into tool-specific input formats (e.g. GraPhlAn tree.txt
  and annot.txt) via shell-invoked CLI tools.
when_to_use_negative:
- The goal is interactive exploration of a single phylogeny with overlaid metadata,
  where iTOL or Krona may be more appropriate
- Input data are raw reads or unprocessed profiles rather than MetaPhlAn/LEfSe/HUMAnN
  outputs already in LEfSe input format
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: export2graphlan
  role: Converts MetaPhlAn, LEfSe, and/or HUMAnN output into the GraPhlAn tree and
    annotation files
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated tree into a high-quality, compact circular phylogeny
    image
  repo: https://bitbucket.org/nsegata/graphlan
- name: Galaxy
  role: Alternative web-based execution environment for GraPhlAn within pipelines
- name: matplotlib
  role: Underlying plotting library used by GraPhlAn for rendering
- name: hclust2
  role: Companion command-line tool (SciPy dependency) for heatmap visualization of
    MetaPhlAn2 profiles in the same pipeline ecosystem
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:8848694dd9540030100595225c54e805b3e27f7497cb6ae627216a4bd676c7e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - Galaxy
  - GraPhlAn
  - export2graphlan
  - hclust2
  - matplotlib
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# command-line-bioinformatics-pipelining

## Summary

Chains command-driven bioinformatics tools (export2graphlan, graphlan_annotate.py, graphlan.py) into a reproducible pipeline that converts MetaPhlAn/LEfSe/HUMAnN outputs into GraPhlAn tree and annotation files and renders the final figure.

## When to use

Use when a pipeline must convert metagenomic profiler outputs (MetaPhlAn v1/v2, HUMAnN, LEfSe results) into tool-specific input formats (e.g. GraPhlAn tree.txt and annot.txt) via shell-invoked CLI tools.

## When NOT to use

- The goal is interactive exploration of a single phylogeny with overlaid metadata, where iTOL or Krona may be more appropriate
- Input data are raw reads or unprocessed profiles rather than MetaPhlAn/LEfSe/HUMAnN outputs already in LEfSe input format

## Inputs

- MetaPhlAn (v1 or v2) merged abundance profile in LEfSe input format (e.g. hmp_metahit.txt)
- LEfSe result file (biomarker LDA scores)
- Optionally HUMAnN functional profiles (KEGG ortholog groups)
- Optionally BIOM-format abundance tables (requires biom-format ver. 2.0.1)

## Outputs

- tree.txt — GraPhlAn tree file representing all input clades
- annot.txt — GraPhlAn annotation file with ring colors/highlighting for LEfSe biomarker clades
- outtree.txt — annotated tree produced by graphlan_annotate.py
- Final rendered GraPhlAn image

## How to apply

Write a shell driver (e.g. PIPELINE.sh) that first runs export2graphlan.py with -i/--lefse_input (merged MetaPhlAn abundance matrix in LEfSe format) and -o/--lefse_output (LEfSe biomarker result), writing -t tree.txt and -a annot.txt. Set clade sizing so non-biomarker clades get --def_clade_size 10 while LEfSe biomarkers scale between --min_clade_size 20 and --max_clade_size 200, with font sizes defaulting to 10/8/12; pass --internal_levels to sum abundances leaf-to-root on internal nodes, and optionally restrict annotation by --abundance_threshold 20.0 and --most_abundant 10. Then run graphlan_annotate.py to attach the annotations to the tree (producing outtree.txt), and finally graphlan.py to render the image. The rationale, per GraPhlAn, is that visualization tools must be command-driven so they embed in complex, publication-quality bioinformatics pipelines.

## Related tools

- **export2graphlan** (Converts MetaPhlAn, LEfSe, and/or HUMAnN output into the GraPhlAn tree and annotation files) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated tree into a high-quality, compact circular phylogeny image) — https://bitbucket.org/nsegata/graphlan
- **Galaxy** (Alternative web-based execution environment for GraPhlAn within pipelines)
- **matplotlib** (Underlying plotting library used by GraPhlAn for rendering)
- **hclust2** (Companion command-line tool (SciPy dependency) for heatmap visualization of MetaPhlAn2 profiles in the same pipeline ecosystem) — https://github.com/SegataLab/hclust2

## Examples

```
export2graphlan.py -i lefse_input.txt -o lefse_output.txt -t tree.txt -a annot.txt --def_clade_size 10 --min_clade_size 20 --max_clade_size 200 --internal_levels
```

## Evaluation signals

- Both tree.txt and annot.txt exist and are non-empty after export2graphlan.py runs
- tree.txt parses as a GraPhlAn tree with all input clades represented
- annot.txt contains annotation entries (ring colors/highlighting) for the LEfSe biomarker clades listed in the LEfSe result file
- Non-biomarker clade sizes and font sizes match the defaults (def_clade_size 10; fonts 10/8/12) while biomarker clades fall within 20–200
- The full PIPELINE.sh execution completes and yields an annotated tree (outtree.txt) plus a rendered image

## Limitations

- No software version numbers stated for GraPhlAn, export2graphlan, or its dependencies (matplotlib, pandas 0.13.1, BIOM 2.0.1); dependency pinning must be handled externally
- export2graphlan requires additional libraries (pandas ver. 0.13.1, BIOM ver. 2.0.1 for BIOM inputs, SciPy for hclust2)
- When LEfSe biomarker labels are absent, annotation falls back to abundance thresholds (default 20.0) and the most abundant clades (default 10), which may not reflect biomarker-based highlighting

## Evidence

- [methods] Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format.: "Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format."
- [abstract] Pipeline embedding rationale: "GraPhlAn has been developed as an open-source command-driven tool in order to be easily integrated into complex, publication-quality bioinformatics pipelines."
- [readme] export2graphlan purpose from README: "export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn."
- [readme] Clade sizing defaults from README usage: "Set a default size for clades that are not found as biomarkers by LEfSe. Default is 10"
- [methods] Internal levels option: "--internal_levels     If specified sum-up from leaf to root the abundances values."
