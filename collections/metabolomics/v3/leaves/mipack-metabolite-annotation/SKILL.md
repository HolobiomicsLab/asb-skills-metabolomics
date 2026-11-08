---
name: mipack-metabolite-annotation
description: Use when a processed, aligned peak list from the Galaxy-M DIMS SIM-stitch
  pipeline (calibrated m/z values with blank/replicate filtering already applied)
  needs metabolite annotation as the final analysis step of the workflow.
when_to_use_negative:
- Input is still raw instrument data (.dat transient files) rather than a processed,
  calibrated peak list — the SimStitch/alignment/filtering steps must run first
- The task requires statistical testing (t-test p-values, PCA Scores Test) rather
  than annotation — Galaxy-M's PLS-Toolbox/PCA tools cover that, and no univariate
  statistics tools are included in the current implementation
- Peak detection itself is needed — MI-Pack annotates already-detected peaks; peak
  picking is done upstream by SIM-stitching (DIMS) or XCMS (LC-MS)
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: MI-Pack
  role: 'Core annotation engine: performs metabolite identification from the Galaxy-M
    peak list against compound reference databases and exports the tab-delimited annotation
    file'
  repo: https://github.com/Viant-Metabolomics/MI-Pack
- name: Galaxy-M
  role: Hosts the workflow in which MI-Pack runs as the final annotation step; Galaxy-M
    wraps MI-Pack tool files (.xml wrappers) and supplies the upstream processed peak
    list
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:e70ab9cb26da9caf4d352d3f36b984e33e07ccb8a9f2d8d995e5652dba885ed4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/MI-Pack
  tools:
  - Galaxy-M
  - MI-Pack
  - PLS-Toolbox
  - SimStitch
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# mipack-metabolite-annotation

## Summary

MI-Pack annotates peaks detected by the Galaxy-M DIMS/LC-MS processing pipelines against compound reference databases (KEGG, HMDB, LIPID MAPS) by integrating accurate masses and metabolic pathways, exporting the results as a tab-delimited annotation table.

## When to use

Use when a processed, aligned peak list from the Galaxy-M DIMS SIM-stitch pipeline (calibrated m/z values with blank/replicate filtering already applied) needs metabolite annotation as the final analysis step of the workflow.

## When NOT to use

- Input is still raw instrument data (.dat transient files) rather than a processed, calibrated peak list — the SimStitch/alignment/filtering steps must run first
- The task requires statistical testing (t-test p-values, PCA Scores Test) rather than annotation — Galaxy-M's PLS-Toolbox/PCA tools cover that, and no univariate statistics tools are included in the current implementation
- Peak detection itself is needed — MI-Pack annotates already-detected peaks; peak picking is done upstream by SIM-stitching (DIMS) or XCMS (LC-MS)

## Inputs

- Aligned, filtered peak list (peak m/z values across samples) produced by the Galaxy-M DIMS SIM-stitch pipeline (Sum Transients, Process Transients, Mass Calibration and SIM-stitching, Align Samples, then replicate/blank/sample filtering)
- Compound reference databases (KEGG, HMDB, LIPID MAPS)

## Outputs

- Tab-delimited metabolite annotation file mapping detected peaks to putative metabolites
- In the DIMS case study, annotations equivalent to those tabulated in Additional file 1: Table S2 of the publication

## How to apply

Run MI-Pack as the last stage of the Galaxy-M DIMS history, after Align Samples, Replicate Filtering, Blank Filtering, Sample Filtering, missing value imputation, normalization and scaling have produced a cleaned peak table. MI-Pack takes the detected peak m/z values and searches them against compound reference databases such as KEGG, HMDB and LIPID MAPS, using accurate-mass matching combined with metabolic pathway context to increase confidence of metabolite identification. In the published DIMS case study (MTBLS79 cow vs sheep heart subset), the MI-Pack output is a tab-delimited file of metabolite annotations that should reproduce the annotations stored in the 'DIMS Test Data processing' Galaxy history. For reproducibility, use the MI-Pack GitHub commit 06ce0ace643ee1cf1d27550769a5272b7ea50825 that the GigaScience publication utilises. Install the package via python setup.py build followed by python setup.py install; optional parallel python (v1.6.4) speeds up computationally heavy annotation steps.

## Related tools

- **MI-Pack** (Core annotation engine: performs metabolite identification from the Galaxy-M peak list against compound reference databases and exports the tab-delimited annotation file) — https://github.com/Viant-Metabolomics/MI-Pack
- **Galaxy-M** (Hosts the workflow in which MI-Pack runs as the final annotation step; Galaxy-M wraps MI-Pack tool files (.xml wrappers) and supplies the upstream processed peak list) — https://github.com/Viant-Metabolomics/Galaxy-M

## Examples

```
python setup.py build; python setup.py install
```

## Evaluation signals

- The Galaxy history completes and produces the MI-Pack metabolite annotation tab-delimited file without tool errors
- Annotation results reproduce the metabolite annotations stored in the published 'DIMS Test Data processing' Galaxy history on the Galaxy-M VM for the MTBLS79 subset
- Annotated metabolites match those tabulated in Additional file 1: Table S2 of the GigaScience publication
- The MI-Pack installation matches the pinned commit 06ce0ace643ee1cf1d27550769a5272b7ea50825 used for the publication, ensuring identical database-search behaviour

## Limitations

- Reproducibility depends on the specific MI-Pack GitHub commit (06ce0ace643ee1cf1d27550769a5272b7ea50825); other versions may give different annotations
- MI-Pack was used in a Python 2.7 environment with optional parallel python 1.6.4; other runtime versions are not guaranteed to be compatible
- DIMS transient (.dat) input files are not in the MetaboLights MTBLS79 accession and must be obtained from the GigaDB VM/original authors before the upstream pipeline can feed MI-Pack
- MI-Pack provides identification only; the Galaxy-M implementation includes no univariate statistics tools and the PCA tool has no graphical output

## Evidence

- [abstract] Metabolite annotation is performed using MI-Pack [10].: "Metabolite annotation is performed using MI-Pack [10]."
- [methods] Download the MI-Pack python package from https://github.com/Viant-Metabolomics/MI-Pack: "Download the MI-Pack python package from https://github.com/Viant-Metabolomics/MI-Pack"
- [abstract] compound reference databases (e.g. KEGG, HMDB and LIPID MAPS): "compound reference databases (e.g. KEGG, HMDB and LIPID MAPS)"
- [readme] MI-Pack: Increased confidence of metabolite identification in mass spectra by integrating accurate masses and metabolic pathways: "MI-Pack: Increased confidence of metabolite identification in mass spectra by integrating accurate masses and metabolic pathways"
- [readme] The MI-Pack version that the GigaScience publication utilises is Github commit: 06ce0ace643ee1cf1d27550769a5272b7ea50825: "The MI-Pack version that the GigaScience publication utilises is Github commit: 06ce0ace643ee1cf1d27550769a5272b7ea50825"
