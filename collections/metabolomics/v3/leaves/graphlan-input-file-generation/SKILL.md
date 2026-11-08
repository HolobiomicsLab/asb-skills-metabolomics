---
name: graphlan-input-file-generation
description: Use when you have MetaPhlAn (v1/v2), HUMAnN, or BIOM-format (v1/v2, e.g.
  QIIME/mothur 16S) profiles plus optionally a LEfSe result, and need to produce GraPhlAn
  tree.txt and annot.txt for circular phylogenetic visualization.
when_to_use_negative:
- Your tree data is already in GraPhlAn tree/annotation format and needs no conversion
  from MetaPhlAn/LEfSe/HUMAnN or BIOM outputs
- You need interactive exploration of large phylogenies rather than static publication
  figures — iTOL or Krona target interactive analyses instead
- The tree contains no abundance, LEfSe, or metadata annotation to encode, so automatic
  biomarker highlighting has nothing to act on
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3174
tools:
- name: export2graphlan
  role: Converts MetaPhlAn, LEfSe, and/or HUMAnN outputs (or BIOM tables) into the
    GraPhlAn tree file and annotation file
  repo: https://github.com/SegataLab/export2graphlan
- name: GraPhlAn
  role: Renders the annotated circular cladogram via graphlan_annotate.py and graphlan.py
    using the matplotlib library
  repo: https://bitbucket.org/nsegata/graphlan
- name: matplotlib
  role: Underlying plotting library used by GraPhlAn to draw the circular tree images
- name: hclust2
  role: Companion SegataLab tool for heat-map visualization of abundance profiles
    (e.g. MetaPhlAn2 HMP/HMP1-phase2 clustered profiles)
  repo: https://github.com/SegataLab/hclust2
- name: Galaxy
  role: Web interface through which GraPhlAn can be executed online instead of locally
provenance:
  source_task_ids:
  - task_002
  - task_008
  - task_001
  source_papers:
  - doi: 10.7717/peerj.1029
    title: Compact graphical representation of phylogenetic data and metadata with
      GraPhlAn
  source_content_hash: sha256:5ddc1fe9a9f52e9373db6f90f5363e004475a7cd7e65dd25a79afbb5dd07e39a
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
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# graphlan-input-file-generation

## Summary

Generates the two GraPhlAn input files (tree file and annotation file) from MetaPhlAn/LEfSe/HUMAnN results or BIOM-format abundance tables using export2graphlan, automatically highlighting LEfSe biomarker clades and abundant taxa before rendering with graphlan_annotate.py and GraPhlAn.

## When to use

Use when you have MetaPhlAn (v1/v2), HUMAnN, or BIOM-format (v1/v2, e.g. QIIME/mothur 16S) profiles plus optionally a LEfSe result, and need to produce GraPhlAn tree.txt and annot.txt for circular phylogenetic visualization.

## When NOT to use

- Your tree data is already in GraPhlAn tree/annotation format and needs no conversion from MetaPhlAn/LEfSe/HUMAnN or BIOM outputs
- You need interactive exploration of large phylogenies rather than static publication figures — iTOL or Krona target interactive analyses instead
- The tree contains no abundance, LEfSe, or metadata annotation to encode, so automatic biomarker highlighting has nothing to act on

## Inputs

- LEfSe-format merged abundance matrix (e.g. MetaPhlAn profile merge, -i/--lefse_input)
- LEfSe result file with LDA biomarker scores (-o/--lefse_output)
- BIOM-format abundance table (versions 1 and 2), e.g. from QIIME or mothur 16S studies
- HUMAnN functional profiling results

## Outputs

- tree.txt (GraPhlAn tree file with all input clades represented)
- annot.txt (GraPhlAn annotation file with ring colors/highlighting for LEfSe biomarker clades)
- outtree.txt (annotated tree produced by graphlan_annotate.py)
- Rendered circular tree image (e.g. outimg.png) plus annotation-legend and highlighted-biomarker legend images

## How to apply

Run export2graphlan.py with the LEfSe-format abundance matrix as -i/--lefse_input and the LEfSe result as -o/--lefse_output, emitting -t tree.txt and -a annot.txt. Clades not flagged as LEfSe biomarkers receive the default clade size (--def_clade_size 10) while biomarker clades scale between --min_clade_size 20 and --max_clade_size 200, with default/min/max font sizes 10/8/12; annotation is restricted to clades passing --abundance_threshold (default 20.0) and the --most_abundant clades (default 10) when biomarker labels are absent. Enable --internal_levels to sum abundance values from leaf to root on internal taxonomy nodes before highlighting. BIOM versions 1 and 2 are accepted (requires biom-format 2.0.1, pandas 0.13.1, SciPy), enabling input from QIIME and mothur frameworks such as 16S vaginal microbiota profiles. Then annotate the tree with graphlan_annotate.py to produce outtree.txt and render it with GraPhlAn (matplotlib-based) to the output image plus annotation and biomarker legend images.

## Related tools

- **export2graphlan** (Converts MetaPhlAn, LEfSe, and/or HUMAnN outputs (or BIOM tables) into the GraPhlAn tree file and annotation file) — https://github.com/SegataLab/export2graphlan
- **GraPhlAn** (Renders the annotated circular cladogram via graphlan_annotate.py and graphlan.py using the matplotlib library) — https://bitbucket.org/nsegata/graphlan
- **matplotlib** (Underlying plotting library used by GraPhlAn to draw the circular tree images)
- **hclust2** (Companion SegataLab tool for heat-map visualization of abundance profiles (e.g. MetaPhlAn2 HMP/HMP1-phase2 clustered profiles)) — https://github.com/SegataLab/hclust2
- **Galaxy** (Web interface through which GraPhlAn can be executed online instead of locally)

## Examples

```
export2graphlan.py -i lefse_input.txt -o lefse_output.txt -t tree.txt -a annot.txt --internal_levels --def_clade_size 10 --min_clade_size 20 --max_clade_size 200
```

## Evaluation signals

- Both tree.txt and annot.txt exist and are non-empty after the export2graphlan run
- tree.txt parses as a GraPhlAn tree with all input clades represented
- annot.txt contains annotation entries (ring colors/highlighting) for every LEfSe biomarker clade listed in the LEfSe result file
- The full render produces the six expected files (annot.txt, outimg.png, outimg_annot.png, outimg_legend.png, outtree.txt, tree.txt) for the BIOM-based example
- The rendered outimg.png displays a circular tree consistent with the reference figure (e.g. the Fig. S3 vaginal microbiota temporal-dynamics representation)

## Limitations

- No software version numbers are stated for GraPhlAn itself; export2graphlan requires specific dependencies (pandas 0.13.1, BIOM 2.0.1 for BIOM inputs, SciPy)
- No changelog is available in the repository documentation
- Clades absent from the LEfSe biomarker list are only annotated if they pass the abundance threshold (default 20.0) and fall within the most abundant clades (default 10)

## Evidence

- [readme] export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn.: "export2graphlan is a conversion software tool for producing both annotation and tree file for GraPhlAn. In particular, the annotation file tries to highlight specific sub-trees"
- [readme] BIOM format acceptance: "BIOM ver. 2.0.1 ([biom-format](http://biom-format.org), only if you have input files in BIOM format)"
- [methods] internal levels flag: "--internal_levels     If specified sum-up from leaf to root the abundances values."
- [methods] annotated tree step: "``outtree.txt``: is the annotated tree produced by graphlan_annotate.py"
- [discussion] matplotlib rendering: "GraPhlAn uses the matplotlib library (Hunter, 2007)."
