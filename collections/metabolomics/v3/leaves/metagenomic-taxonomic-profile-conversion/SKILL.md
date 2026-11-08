---
name: metagenomic-taxonomic-profile-conversion
description: Use when you have a LEfSe-format abundance matrix (from merged MetaPhlAn
  profiles) and/or a LEfSe biomarker result file and need to render them as a GraPhlAn
  circular phylogeny with clade-size and ring annotations.
when_to_use_negative:
- Input is already a GraPhlAn tree/annotation pair — run graphlan_annotate.py and
  graphlan.py directly instead
- Input is raw sequencing reads or an unmerged MetaPhlAn per-sample profile lacking
  the LEfSe hierarchical taxonomy format
- The goal is interactive exploration of large phylogenies with metadata overlays
  rather than publication-quality circular plots (e.g., consider iTOL or Krona)
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: export2graphlan
  role: 'Core converter: reads MetaPhlAn/LEfSe (and HUMAnN) outputs and writes the
    GraPhlAn tree and annotation files'
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: 'Downstream renderer: consumes the produced tree and annotation files to draw
    the circular phylogeny'
  repo: https://bitbucket.org/nsegata/graphlan
- name: hclust2
  role: Dependency used by export2graphlan for table handling / hierarchical clustering
    of abundance matrices
  repo: https://github.com/SegataLab/hclust2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:7dd790a5ea7c16c163b08a42242b22248f81c978f13f13245a802cfda2e25077
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - GraPhlAn
  - export2graphlan
  - hclust2
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metagenomic-taxonomic-profile-conversion

## Summary

Converts MetaPhlAn (v1/v2), HUMAnN, and LEfSe outputs into the two input files (tree file and annotation file) required by GraPhlAn, using export2graphlan.py to annotate and highlight the most abundant clades and LEfSe biomarker clades.

## When to use

Use when you have a LEfSe-format abundance matrix (from merged MetaPhlAn profiles) and/or a LEfSe biomarker result file and need to render them as a GraPhlAn circular phylogeny with clade-size and ring annotations.

## When NOT to use

- Input is already a GraPhlAn tree/annotation pair — run graphlan_annotate.py and graphlan.py directly instead
- Input is raw sequencing reads or an unmerged MetaPhlAn per-sample profile lacking the LEfSe hierarchical taxonomy format
- The goal is interactive exploration of large phylogenies with metadata overlays rather than publication-quality circular plots (e.g., consider iTOL or Krona)

## Inputs

- LEfSe-format abundance matrix produced by merging MetaPhlAn profiles (or BIOM-format abundance table)
- LEfSe result file listing biomarkers with LDA scores (as produced by LEfSe on the abundance matrix)
- Optionally, HUMAnN functional profiling results with KEGG ortholog groups

## Outputs

- tree.txt: GraPhlAn tree file with all input clades represented
- annot.txt: GraPhlAn annotation file with clade sizes, font sizes, and highlight/ring entries for LEfSe biomarker clades

## How to apply

Load the LEfSe-format abundance matrix as the -i/--lefse_input argument and the LEfSe result file as -o/--lefse_output of export2graphlan.py, then run it with -t tree.txt and -a annot.txt so that clades not flagged as biomarkers get --def_clade_size 10 while biomarker clades scale between --min_clade_size 20 and --max_clade_size 200, with default/min/max font sizes of 10/8/12. Enable --internal_levels to sum abundances from leaf to root on internal taxonomy nodes before highlighting. Restrict annotation to clades passing the abundance threshold (default 20.0) and, when biomarker labels are absent, to the most abundant clades (default 10). The resulting files are consumed downstream by graphlan_annotate.py (to attach annotations) and graphlan.py (to render the image); export2graphlan requires pandas 0.13.1, BIOM 2.0.1 (only for BIOM-format inputs), and SciPy.

## Related tools

- **export2graphlan** (Core converter: reads MetaPhlAn/LEfSe (and HUMAnN) outputs and writes the GraPhlAn tree and annotation files) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Downstream renderer: consumes the produced tree and annotation files to draw the circular phylogeny) — https://bitbucket.org/nsegata/graphlan
- **hclust2** (Dependency used by export2graphlan for table handling / hierarchical clustering of abundance matrices) — https://github.com/SegataLab/hclust2

## Examples

```
export2graphlan.py -i lefse_input.txt -o lefse_output.txt -t tree.txt -a annot.txt --internal_levels --def_clade_size 10 --min_clade_size 20 --max_clade_size 200
```

## Evaluation signals

- Both tree.txt and annot.txt exist and are non-empty after the run
- tree.txt parses as a GraPhlAn tree and every clade from the input LEfSe-format matrix is represented
- annot.txt contains annotation entries (ring colors/highlighting) for each LEfSe biomarker clade listed in the LEfSe result file
- Clade sizes in annot.txt fall in the expected ranges: default 10 for non-biomarkers, 20–200 for biomarker clades
- With --internal_levels enabled, internal-node abundances equal the sum of descendant leaf abundances (leaf-to-root)

## Limitations

- No software version numbers stated for GraPhlAn or dependencies beyond pandas 0.13.1 and BIOM 2.0.1; version drift may affect behavior
- BIOM support requires an additional library install (biom-format) and only applies when inputs are in BIOM format
- Without LEfSe biomarker labels, annotation falls back to an abundance threshold (default 20.0) and the most abundant clades (default 10), which may omit rare but biologically relevant clades

## Evidence

- [methods] Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format: "Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format."
- [methods] internal levels sum abundances: "--internal_levels     If specified sum-up from leaf to root the abundances values."
- [readme] export2graphlan purpose (README): "export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn."
- [readme] default clade-size parameters (README): "Set a default size for clades that are not found as biomarkers by LEfSe. Default is 10"
- [readme] downstream consumption of outputs (README): "The two output file of export2graphlan should then be used to run graphlan_annotate.py, in order to attach to the tree the derived annotations"
