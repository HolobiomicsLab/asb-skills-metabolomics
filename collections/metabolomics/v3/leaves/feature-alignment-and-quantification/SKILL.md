---
name: feature-alignment-and-quantification
description: Use when raw centroided LC-MS mzML runs (e.g., from MassIVE accessions
  like MSV000080560 or MSV000080553) must be converted into an aligned feature quantification
  table as the first computational stage of untargeted metabolomics or molecular cartography.
when_to_use_negative:
- Input is already a feature quantification table or peak-picked feature list — alignment/detection
  would be redundant
- Data are direct-infusion or non-chromatographic runs without a shared RT scale requiring
  gradient-specific RT tolerance
- The goal is MS/MS spectral library matching only, without spatial or comparative
  quantification across runs
edam_operation: http://edamontology.org/operation_3628
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: Optimus
  role: KNIME workflow hosting the 'Detect LC-MS features' and 'Align and quantify
    features' nodes
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: provides the state-of-the-art LC-MS feature detection and quantification algorithms
    underlying the Optimus nodes
- name: KNIME
  role: workflow management platform in which the detection/alignment pipeline is
    configured and executed
- name: OptimusViewer
  role: visualizes the feature quantification matrix and XIC/MS/MS plots produced
    by the workflow
  repo: https://github.com/alexandrovteam/OptimusViewer
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:1b28a778669745a3d530167cedf8618e2d68989b0a29f0f85b84a5a8549b4510
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MolecularCartography/Optimus
  tools:
  - '''ili'
  - KNIME
  - OpenMS
  - Optimus
  - OptimusViewer
  - Python
  - pyopenms
  edam_operation: http://edamontology.org/operation_3628
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# feature-alignment-and-quantification

## Summary

Detect LC-MS features in each centroided mzML run with OpenMS-based nodes inside the Optimus KNIME workflow, then align and quantify features across all runs to build a sample-by-feature intensity matrix that feeds downstream filtering, annotation, and 'ili spatial mapping.

## When to use

Use when raw centroided LC-MS mzML runs (e.g., from MassIVE accessions like MSV000080560 or MSV000080553) must be converted into an aligned feature quantification table as the first computational stage of untargeted metabolomics or molecular cartography.

## When NOT to use

- Input is already a feature quantification table or peak-picked feature list — alignment/detection would be redundant
- Data are direct-infusion or non-chromatographic runs without a shared RT scale requiring gradient-specific RT tolerance
- The goal is MS/MS spectral library matching only, without spatial or comparative quantification across runs

## Inputs

- centroided mzML files for all LC-MS runs (samples, blanks, QCs, replicates)
- optimus_experimental_design.csv marking run types (BLANK, QC, sample)
- Optimus KNIME workflow configuration (instrument type: QTOF or Orbitrap)

## Outputs

- features_quantification_matrix.csv (features × runs intensity matrix)
- OptimusViewer_input.db (feature database with m/z, RT, charge, annotations)
- spectra_ms2.mgf (MS/MS spectra of detected features)
- Optimus_settings.ini (workflow parameter record)

## How to apply

Load centroided mzML files into the Optimus 'Read LC-MS runs' node in KNIME and mark blanks/QCs in optimus_experimental_design.csv. Configure the OpenMS-based 'Detect LC-MS features' node with m/z tolerance 20 p.p.m. for QTOF instruments (5–10 p.p.m. for Orbitrap) and noise threshold 5,000 for QTOF (100,000–1,000,000 for Orbitrap); these tolerances match instrument mass accuracy so that the same compound is detected as one feature across runs. In the 'Align and quantify features' node set 'RT tolerance (s)' to 30 s (the default for a 10-min UHPLC gradient), which allows for retention-time drift during alignment. Run the workflow to emit features_quantification_matrix.csv, Optimus_settings.ini, OptimusViewer_input.db, and spectra_ms2.mgf; the matrix can then be inspected as a heat map in OptimusViewer or passed to 'ili input generation.

## Related tools

- **Optimus** (KNIME workflow hosting the 'Detect LC-MS features' and 'Align and quantify features' nodes) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (provides the state-of-the-art LC-MS feature detection and quantification algorithms underlying the Optimus nodes)
- **KNIME** (workflow management platform in which the detection/alignment pipeline is configured and executed)
- **OptimusViewer** (visualizes the feature quantification matrix and XIC/MS/MS plots produced by the workflow) — https://github.com/alexandrovteam/OptimusViewer

## Evaluation signals

- Output folder contains features_quantification_matrix.csv, Optimus_settings.ini, OptimusViewer_input.db, and spectra_ms2.mgf
- Number of consensus features is consistent across runs after alignment (no run dropped from the matrix)
- Known internal standards appear at consistent retention times across aligned runs within the 30 s RT tolerance
- OptimusViewer_input.db opens in OptimusViewer and shows m/z, RT, charge, and run columns populated
- Settings recorded in Optimus_settings.ini match the intended tolerances (20 p.p.m. QTOF / 5–10 p.p.m. Orbitrap, RT 30 s, noise 5,000 QTOF)

## Limitations

- Feature detection quality depends on instrument type: m/z tolerance and noise thresholds must be retuned for Orbitrap (5–10 p.p.m., 100,000–1,000,000 noise) versus QTOF (20 p.p.m., 5,000 noise)
- RT tolerance of 30 s assumes a 10-min UHPLC gradient; longer gradients or large RT drift require re-adjustment
- 2 GB RAM is insufficient for datasets of about a hundred or more LC-MS runs; temporary files may occupy several times the raw dataset size and are not deleted automatically

## Evidence

- [methods] workflow card parameters for QTOF/Orbitrap and RT tolerance: [evidence span withheld — anchor status: not_found]
- [methods] task card RT tolerance default: [evidence span withheld — anchor status: not_found]
- [readme] README output artifacts of the workflow: [evidence span withheld — anchor status: external_doc]
