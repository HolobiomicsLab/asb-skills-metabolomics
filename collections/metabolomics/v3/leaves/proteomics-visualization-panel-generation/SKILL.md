---
name: proteomics-visualization-panel-generation
description: Use when processed proteomics results (e.g., deposited dataset PXD037527
  from DDA, DIA, or WWA acquisition on an Orbitrap Exploris 480) must be converted
  into figure panels and checked for concordance with a published manuscript figure.
when_to_use_negative:
- Input is still raw MS data rather than processed results — per the README, 'MSConnect
  is not a standalone data analysis tool. It depends on third-party software for data
  processing/analysis'
- The goal is a new exploratory analysis rather than reproducing/comparing panels
  from an existing published study
- No suitable visualization app exists in the Marketplace for the required statistic
  — the README warns visualization 'depends on user workflow may required 3rd party
  modules'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: MSConnect
  role: Web-based server platform whose Marketplace visualization apps (Python/HTML)
    regenerate the figure panels from the processed PXD037527 results
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: Proteomics_Data_Processor
  role: Upstream companion tool that carries raw files through processing so results
    are traceable into the visualization step
  repo: https://github.com/RTKlab-BYU/Proteomics_Data_Processor
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:f8e08dba6af6ea340f4853713b13b5c76bf3ddb5105d63f491f478b5f4970e30
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - MSConnect
  - PXD037527
  - Proteomics_Data_Processor
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# proteomics-visualization-panel-generation

## Summary

Regenerates publication-ready visualization panels from processed MS-based proteomics results using MSConnect Server's Python/HTML visualization apps, so that outputs can be directly compared against panels in the original study (e.g., Figure 5C panels matching Figure 4 of the PXD037527 single-cell DDA/DIA/WWA manuscript).

## When to use

Use when processed proteomics results (e.g., deposited dataset PXD037527 from DDA, DIA, or WWA acquisition on an Orbitrap Exploris 480) must be converted into figure panels and checked for concordance with a published manuscript figure.

## When NOT to use

- Input is still raw MS data rather than processed results — per the README, 'MSConnect is not a standalone data analysis tool. It depends on third-party software for data processing/analysis'
- The goal is a new exploratory analysis rather than reproducing/comparing panels from an existing published study
- No suitable visualization app exists in the Marketplace for the required statistic — the README warns visualization 'depends on user workflow may required 3rd party modules'

## Inputs

- Processed single-cell proteomics result tables for dataset PXD037527 (DDA, DIA, and WWA acquisition on an Orbitrap Exploris 480) from the deposited repository
- FDR-filtered identification/quantification outputs (PSMs, peptides, protein groups) carried through search programs and postprocessing
- MSConnect visualization app (Python/HTML-based app from the MSConnect Apps Marketplace, e.g., https://proteomicsdata.com)

## Outputs

- Regenerated figure panels (e.g., Figure 5C panels) exported as image files
- Panels quantitatively and visually compared against the original study's Figure 4 to assess concordance

## How to apply

Obtain the processed results for the deposited dataset (PXD037527) from the repository and load them into the MSConnect Server environment, a web-based platform built with Python, Django, JavaScript, and HTML. Install the needed visualization app from the Marketplace; the paper notes that 'installing a wrapper or visualization app is as simple as downloading a new app from a mobile app store.' Run the app's statistical models and plotting routines over the processed tables so raw files are traceably carried through search programs, postprocessing, and visualization. Export each regenerated panel as an image file. Compare the exported panels visually and quantitatively against the original manuscript's Figure 4 to assess concordance; the paper reports MSConnect automatically produced Figure 5C panels matching Figure 4 of the original single-cell proteomics study.

## Related tools

- **MSConnect** (Web-based server platform whose Marketplace visualization apps (Python/HTML) regenerate the figure panels from the processed PXD037527 results) — https://github.com/RTKlab-BYU/MSConnect
- **Proteomics_Data_Processor** (Upstream companion tool that carries raw files through processing so results are traceable into the visualization step) — https://github.com/RTKlab-BYU/Proteomics_Data_Processor

## Evaluation signals

- Each exported panel image matches the corresponding panel of the original manuscript's Figure 4 both visually and quantitatively
- The full chain is traceable: the same raw files are carried through search programs, postprocessing, and visualization (MSConnect's traceable-workflow invariant)
- Tabulated PSM, peptide, and protein-group counts passing the given FDR filter are consistent with the processed results loaded into the app
- Panels are produced without manual data re-entry, consistent with MSConnect's automated end-to-end workflow claim
- Every regenerated panel is exported as a standalone image file suitable for publication use

## Limitations

- MSConnect does not include third-party analysis software or licenses; it provides a wrapper/interface, so visualization depends on user workflow and may require third-party modules
- Experimental design module not yet released, so panel-generating workflows are not yet formally tied to tracked experimental design
- Reproducibility depends on the processed results and workflow description being sufficient — the paper notes manuscript methods sections are often insufficient to ensure reproducibility of complex data analysis strategies
- No changelog found for the repository, so app version drift between the original run and a reproduction is hard to audit

## Evidence
