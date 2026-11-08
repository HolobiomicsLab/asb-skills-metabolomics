---
name: clade-size-abundance-scaling
description: Use when converting MetaPhlAn/LEfSe (or HUMAnN) output into GraPhlAn
  tree and annotation files and you need biomarker clades to be visually emphasized
  by size rather than a uniform node radius.
when_to_use_negative:
- Input is already a GraPhlAn tree/annotation pair with sizes assigned manually —
  re-scaling would overwrite curated node sizes.
- Analysis is purely interactive exploration of abundance matrices (heatmaps/clustering),
  where hclust2 rather than export2graphlan/GraPhlAn is the appropriate tool.
- No LEfSe result or abundance ranking exists to distinguish biomarker from non-biomarker
  clades, so size scaling has no signal to act on.
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0092
tools:
- name: export2graphlan
  role: Converts MetaPhlAn/LEfSe/HUMAnN output into GraPhlAn tree and annotation files,
    applying clade size and font size scaling for biomarker clades
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the scaled circular phylogeny from the tree and annotation files produced
    by export2graphlan
  repo: https://bitbucket.org/nsegata/graphlan
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:3dd26c593a80507866ff82e3f350f7bcd9dfbff12ffa82f35b72aae931b876d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SegataLab/export2graphlan
  tools:
  - GraPhlAn
  - export2graphlan
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# clade-size-abundance-scaling

## Summary

Scales GraPhlAn clade node sizes (and label fonts) so that clades flagged as LEfSe biomarkers grow proportionally with their LDA signal while non-biomarker clades keep a fixed default size, via export2graphlan's --def_clade_size/--min_clade_size/--max_clade_size parameters.

## When to use

Use when converting MetaPhlAn/LEfSe (or HUMAnN) output into GraPhlAn tree and annotation files and you need biomarker clades to be visually emphasized by size rather than a uniform node radius.

## When NOT to use

- Input is already a GraPhlAn tree/annotation pair with sizes assigned manually — re-scaling would overwrite curated node sizes.
- Analysis is purely interactive exploration of abundance matrices (heatmaps/clustering), where hclust2 rather than export2graphlan/GraPhlAn is the appropriate tool.
- No LEfSe result or abundance ranking exists to distinguish biomarker from non-biomarker clades, so size scaling has no signal to act on.

## Inputs

- LEfSe-format abundance matrix produced by merging MetaPhlAn (v1/v2) profiles (export2graphlan -i/--lefse_input)
- LEfSe result file containing biomarker clades and LDA scores (export2graphlan -o/--lefse_output)
- Optionally, BIOM-format abundance tables (requires biom-format ver. 2.0.1)

## Outputs

- GraPhlAn tree file (e.g. tree.txt) with clade sizes encoded per node
- GraPhlAn annotation file (e.g. annot.txt) with highlighting/ring colors for LEfSe biomarker clades

## How to apply

Run export2graphlan.py with the LEfSe-format abundance matrix (-i) and LEfSe result file (-o), writing the tree with -t tree.txt and annotations with -a annot.txt. Clades not flagged as LEfSe biomarkers receive the default clade size (--def_clade_size 10), while biomarker clades are scaled between --min_clade_size 20 and --max_clade_size 200 according to their abundance/LDA signal; label fonts follow the analogous --def_font_size 10, --min_font_size 8, and --max_font_size 12 defaults. Enable --internal_levels so abundance values are summed from leaf to root on internal taxonomy nodes before size and highlight decisions. If biomarker labels are absent, restrict annotation to clades passing --abundance_threshold (default 20.0) or to the --most_abundant clades (default 10). The rationale, from the GraPhlAn paper, is to compactly encode which clades are discriminative biomarkers versus merely abundant within a publication-quality circular phylogeny.

## Related tools

- **export2graphlan** (Converts MetaPhlAn/LEfSe/HUMAnN output into GraPhlAn tree and annotation files, applying clade size and font size scaling for biomarker clades) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the scaled circular phylogeny from the tree and annotation files produced by export2graphlan) — https://bitbucket.org/nsegata/graphlan

## Examples

```
export2graphlan.py -i lefse_input.txt -o lefse_output.txt -t tree.txt -a annot.txt --def_clade_size 10 --min_clade_size 20 --max_clade_size 200 --internal_levels
```

## Evaluation signals

- Both tree.txt and annot.txt exist and are non-empty after the export2graphlan run.
- tree.txt parses as a GraPhlAn tree and all input clades are represented.
- annot.txt contains annotation entries (ring colors/highlighting) for every LEfSe biomarker clade listed in the LEfSe result file.
- Biomarker clades in the tree carry sizes within the [min_clade_size, max_clade_size] range (default 20–200) while non-biomarkers equal def_clade_size (default 10).
- Internal-node abundances match leaf-to-root sums when --internal_levels is enabled.

## Limitations

- No software version numbers are stated for export2graphlan in the article beyond the README's ver. 0.2.1; parameters such as default sizes (10/20/200) may drift across versions.
- When biomarker labels are absent, annotation falls back to abundance thresholds (default 20.0) and the most abundant clades (default 10), which may not reflect differential biomarker status.
- BIOM-format inputs require biom-format ver. 2.0.1; pandas ver. 0.13.1 and SciPy (for hclust2) are additional prerequisites.

## Evidence

- [readme] Default clade size for non-biomarkers and biomarker scaling range: "--def_clade_size DEF_CLADE_SIZE  Set a default size for clades that are not found as biomarkers by LEfSe. Default is 10"
- [readme] Biomarker clade size bounds: "--min_clade_size MIN_CLADE_SIZE  Set the minimum value of clades that are biomarkers. Default is 20"
- [methods] Internal levels summation: "--internal_levels     If specified sum-up from leaf to root the abundances values."
- [methods] export2graphlan purpose: "Convert MetaPhlAn, LEfSe, and/or HUMAnN output to GraPhlAn input format"
- [methods] Annotate tree after conversion: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
