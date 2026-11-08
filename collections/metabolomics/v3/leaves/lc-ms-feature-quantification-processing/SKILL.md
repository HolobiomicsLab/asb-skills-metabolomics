---
name: lc-ms-feature-quantification-processing
description: Use when you have mzML LC–MS/MS runs needing feature detection, quantification,
  filtering, and level-2 putative compound annotation, or when MS/MS spectra must
  be exported (MGF/MAT) for GNPS, Sirius, or MS-FINDER dereplication.
when_to_use_negative:
- Input is already a processed feature table (e.g. an Optimus '*.db' file to be visualized
  — use OptimusViewer instead of re-running quantification)
- The goal is confirmed (level-1) compound identification — mz-RT matching only yields
  MSI level-2 putative annotations, and Optimus does not provide MS/MS validation
  of putative annotations
- Direct structural elucidation from spectra — export MGF/MAT and hand off to Sirius
  or MS-FINDER rather than expecting Optimus to annotate in silico
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Optimus
  role: KNIME workflow performing LC-MS feature detection, alignment, quantification,
    filtering, mz-RT annotation, and MGF/MAT spectral export
  repo: https://github.com/MolecularCartography/Optimus
- name: OpenMS
  role: Provides the state-of-the-art LC-MS feature detection and quantification algorithms
    underlying Optimus
  repo: http://www.openms.de
- name: KNIME
  role: Workflow management system in which the Optimus pipeline (including 'Read
    mz-RT list', 'Match features by mz-RT', 'Prepare results' nodes) is executed
  repo: https://www.knime.org
- name: GNPS
  role: MS/MS dereplication against public spectral libraries; its filtered output
    .tsv serves as the mz-RT annotation list source
  repo: http://gnps.ucsd.edu/
- name: MS-FINDER
  role: Third-party in-silico prediction tool consuming MAT-exported MS/MS spectra
    of detected features
- name: Sirius
  role: Third-party in-silico prediction tool consuming MGF-exported MS/MS spectra
    of detected features
  repo: https://bio.informatik.uni-jena.de/software/sirius/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:9e20b19345f47c5a590554be38e7a5cceb48346908ea735394645d43a3071f3c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MolecularCartography/Optimus
  tools:
  - GNPS
  - KNIME
  - MS-FINDER
  - OpenMS
  - Optimus
  - Sirius
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# lc-ms-feature-quantification-processing

## Summary

Process LC-MS/MS runs end-to-end with the Optimus KNIME workflow: OpenMS-based feature detection and alignment, optional filtering, and Metabolomics Standards Initiative level-2 putative annotation by mz-RT matching against a GNPS-derived or manual list of molecules of interest.

## When to use

Use when you have mzML LC–MS/MS runs needing feature detection, quantification, filtering, and level-2 putative compound annotation, or when MS/MS spectra must be exported (MGF/MAT) for GNPS, Sirius, or MS-FINDER dereplication.

## When NOT to use

- Input is already a processed feature table (e.g. an Optimus '*.db' file to be visualized — use OptimusViewer instead of re-running quantification)
- The goal is confirmed (level-1) compound identification — mz-RT matching only yields MSI level-2 putative annotations, and Optimus does not provide MS/MS validation of putative annotations
- Direct structural elucidation from spectra — export MGF/MAT and hand off to Sirius or MS-FINDER rather than expecting Optimus to annotate in silico

## Inputs

- mzML LC–MS/MS runs (one per sample spot/run)
- Annotation list: GNPS dereplication .tsv (polarity- and 20 p.p.m.-filtered) or a manual CSV with columns m/z, RT in seconds, compound name (RT = -1 if unknown)
- Experimental design file consumed by the Optimus workflow

## Outputs

- annotated_features.csv — feature quantification matrix with an added compound-annotation field for matched features
- Optimus .db feature quantification matrix (features as rows, runs as columns, with feature ID, m/z, retention time, charge, annotations)
- MGF (and optionally MAT) files of exported MS/MS spectra for GNPS/Sirius/MS-FINDER

## How to apply

Load the mzML LC–MS/MS runs into the Optimus KNIME workflow so OpenMS detects and aligns features across all runs, setting 'Half of MS/MS isolation window (Da)' to half the precursor isolation window width so each feature is paired with its MS/MS spectrum. Apply the optional Optimus filters as appropriate: exclude features from blank/control runs, rarely observed features, features without acquired MS/MS spectra, features not reproducible in pooled QC or replicate runs, and compounds eluting at the beginning or end of LC-runs. For annotation, export MS/MS spectra via the 'Prepare results' node ('Save as MGF', and 'Save as MAT' for MS-FINDER; MS1/MS2 noise thresholds of 2,000 and 100 for a maXis II QTOF; for features with several MS/MS spectra the one with the most intense precursor ion is exported), run GNPS dereplication on the mzML files, and keep only hits with the correct polarity and precursor delta m/z within 20 p.p.m. Build the annotation source as either the filtered GNPS .tsv or a manual CSV with m/z, RT in seconds, and compound name (neutral-molecule formula; RT = -1 when unknown), load it into the 'Read mz-RT list' node, and attach hits with 'Match features by mz-RT'. mz-RT matching yields putatively annotated compounds at level 2 of the Metabolomics Standards Initiative; MS/MS validation of individual annotations is left to expert review.

## Related tools

- **Optimus** (KNIME workflow performing LC-MS feature detection, alignment, quantification, filtering, mz-RT annotation, and MGF/MAT spectral export) — https://github.com/MolecularCartography/Optimus
- **OpenMS** (Provides the state-of-the-art LC-MS feature detection and quantification algorithms underlying Optimus) — http://www.openms.de
- **KNIME** (Workflow management system in which the Optimus pipeline (including 'Read mz-RT list', 'Match features by mz-RT', 'Prepare results' nodes) is executed) — https://www.knime.org
- **GNPS** (MS/MS dereplication against public spectral libraries; its filtered output .tsv serves as the mz-RT annotation list source) — http://gnps.ucsd.edu/
- **MS-FINDER** (Third-party in-silico prediction tool consuming MAT-exported MS/MS spectra of detected features)
- **Sirius** (Third-party in-silico prediction tool consuming MGF-exported MS/MS spectra of detected features) — https://bio.informatik.uni-jena.de/software/sirius/

## Examples

```
pip install six pandas protobuf pymspec pyopenms
```

## Evaluation signals

- The output annotated_features.csv exists and is a CSV containing the feature quantification matrix with an added compound-annotation field for matched features
- Every GNPS-derived annotation respects the precursor delta m/z tolerance of 20 p.p.m. (QTOF) and the correct polarity mode
- Features detected with multiple MS/MS spectra have only the spectrum with the most intense precursor ion exported to MGF
- MS1/MS2 noise thresholds in 'Prepare results' are set to 2,000 and 100 (maXis II QTOF) before export
- Annotation-source rows with unknown retention time carry -1 in the RT column; annotation correctness of individual hits is confirmed by expert review

## Limitations

- MS/MS validation of putative annotations is not provided in Optimus and must be done manually
- mz-RT matching yields only Metabolomics Standards Initiative level-2 (putatively annotated compounds) identifications
- Optimus creates temporary files that can occupy up to a few times the initial dataset size and are not deleted automatically; large datasets (~100+ LC-MS runs) need well beyond the 2 GB minimum RAM

## Evidence
