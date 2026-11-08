---
name: msms-spectral-library-matching
description: Use when detected LC-MS features with acquired MS/MS spectra (mzML runs
  processed in Optimus) need putative molecular annotations via MS/MS library matching
  on GNPS, or mz-RT matching to a GNPS-exported molecule list.
when_to_use_negative:
- When you need Metabolomics Standards Initiative level-1 or validated identifications
  — Optimus only provides level-2 putative annotations via mz-RT matching and MS/MS
  validation is not provided in the workflow
- When the input is already a fully annotated feature table or the annotation list
  comes from an in-house library unrelated to GNPS spectral-library matching output
- When features have no acquired MS/MS spectra — such features are typically excluded
  upstream (step 6 of the Optimus workflow) and cannot be dereplicated on GNPS
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: GNPS
  role: Hosts the public spectral libraries; mzML files are uploaded and a Dereplication
    job performs MS/MS matching, whose output supplies the mz-RT list of molecules
    of interest
- name: Optimus
  role: KNIME workflow that pairs features with MS/MS spectra, exports MGF/MAT via
    the 'Prepare results' node, and re-attaches annotations through 'Read mz-RT list'
    and 'Match features by mz-RT' nodes
  repo: https://github.com/MolecularCartography/Optimus
- name: MS-FINDER
  role: Third-party tool for downstream in-silico analysis consuming the MAT-format
    spectral export
- name: Sirius
  role: Third-party tool for downstream in-silico prediction consuming the MGF-format
    spectral export
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:b7a5ace722f945a0d98bb67aa32222ebfa2a8de3f5ccff778808b2e02a369be4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MolecularCartography/Optimus
  tools:
  - GNPS
  - KNIME
  - MS-FINDER
  - Optimus
  - Sirius
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# MS/MS spectral library matching (GNPS dereplication)

## Summary

Attaches level-2 'putatively annotated compound' metabolite identifications to LC-MS features by exporting feature MS/MS spectra (MGF/MAT) from Optimus, matching them against GNPS public spectral libraries, and feeding the filtered mz-RT list back into Optimus annotation.

## When to use

Use when detected LC-MS features with acquired MS/MS spectra (mzML runs processed in Optimus) need putative molecular annotations via MS/MS library matching on GNPS, or mz-RT matching to a GNPS-exported molecule list.

## When NOT to use

- When you need Metabolomics Standards Initiative level-1 or validated identifications — Optimus only provides level-2 putative annotations via mz-RT matching and MS/MS validation is not provided in the workflow
- When the input is already a fully annotated feature table or the annotation list comes from an in-house library unrelated to GNPS spectral-library matching output
- When features have no acquired MS/MS spectra — such features are typically excluded upstream (step 6 of the Optimus workflow) and cannot be dereplicated on GNPS

## Inputs

- mzML LC-MS/MS runs (e.g., from a maXis II QTOF instrument)
- Optimus-detected and aligned feature quantification matrix with paired MS/MS spectra
- GNPS dereplication output (.tsv), filtered for polarity mode and ≤20 p.p.m. precursor delta m/z
- Alternative annotation source: manually created CSV with three columns (m/z, RT in seconds, Compound name)

## Outputs

- annotated_features.csv — feature quantification matrix with an added compound-annotation field for matched features
- MGF file of feature MS/MS spectra for GNPS (and MAT for MS-FINDER/Sirius workflows)
- Filtered GNPS .tsv of spectral-library hits used as the mz-RT annotation list

## How to apply

Detect and align features in the mzML runs using the Optimus KNIME workflow with OpenMS, setting 'Half of MS/MS isolation window (Da)' to the precursor isolation half-width so each feature is paired with its MS/MS spectrum. Enable MS/MS export in the 'Prepare results' node via 'Save as MGF' (and 'Save as MAT' for MS-FINDER), setting MS1/MS2 noise thresholds to 2,000 and 100 for a maXis II QTOF; when a feature has several MS/MS spectra, export the one with the most intense precursor ion. Upload the mzML files to GNPS and run a Dereplication job against public spectral libraries, then filter the output keeping only hits with the correct polarity mode and a precursor delta m/z within 20 p.p.m. (QTOF). Either use the filtered GNPS .tsv or build a manual CSV with three columns (m/z, RT in seconds, Compound name) — giving neutral-molecule formulas and -1 in the RT column when retention time is unknown — load it into the 'Read mz-RT list' node, and attach hits to detected features via 'Match features by mz-RT'. This realizes Metabolomics Standards Initiative level-2 annotation; MS/MS validation of putative annotations is not provided in Optimus and individual hits need expert review.

## Related tools

- **GNPS** (Hosts the public spectral libraries; mzML files are uploaded and a Dereplication job performs MS/MS matching, whose output supplies the mz-RT list of molecules of interest)
- **Optimus** (KNIME workflow that pairs features with MS/MS spectra, exports MGF/MAT via the 'Prepare results' node, and re-attaches annotations through 'Read mz-RT list' and 'Match features by mz-RT' nodes) — https://github.com/MolecularCartography/Optimus
- **MS-FINDER** (Third-party tool for downstream in-silico analysis consuming the MAT-format spectral export)
- **Sirius** (Third-party tool for downstream in-silico prediction consuming the MGF-format spectral export)

## Evaluation signals

- annotated_features.csv exists, is a valid CSV, and is the feature quantification matrix with an added compound-annotation field for matched features
- Every GNPS-derived annotation respects the ≤20 p.p.m. precursor delta m/z tolerance and the correct polarity mode
- MS1/MS2 noise thresholds in the 'Prepare results' export were set to 2,000 and 100 respectively (maXis II QTOF)
- For features with multiple MS/MS spectra, the exported spectrum is the one with the most intense precursor ion
- Annotation file format is correct: three columns (m/z, RT in seconds, Compound name), neutral-molecule formulas, and -1 in the RT column where retention time is unknown

## Limitations

- Annotations are Metabolomics Standards Initiative level-2 putative identifications; MS/MS validation of putative annotations is needed and is currently not provided in Optimus
- Annotation correctness of individual hits is subject to expert review
- The 20 p.p.m. precursor tolerance is QTOF-specific; other instruments may need different thresholds

## Evidence

- [intro] Level-2 annotation claim in protocol intro: [evidence span withheld — anchor status: not_found]
