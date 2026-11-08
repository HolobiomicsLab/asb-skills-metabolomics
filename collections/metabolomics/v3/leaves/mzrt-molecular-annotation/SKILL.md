---
name: mzrt-molecular-annotation
description: Use when an aligned LC-MS feature quantification matrix exists and detected
  consensus features must be putatively identified by mz-RT matching against a molecule
  list (e.g., GNPS MS/MS matches or a manually built CSV).
when_to_use_negative:
- You already have validated MS/MS spectral identifications and only need quantification/spatial
  mapping — annotation at MSI level 2 adds no confidence beyond what you have.
- Input is raw vendor/mzML runs with no aligned feature table; run OpenMS feature
  detection and alignment in Optimus first.
- Direct-infusion data without retention-time information, where RT matching is not
  meaningful.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: Optimus
  role: KNIME workflow performing putative molecular annotation of detected features
    by mz-RT matching to a list of molecules of interest
  repo: https://github.com/MolecularCartography/Optimus
- name: GNPS
  role: source of the molecules-of-interest list exported from MS/MS matching against
    GNPS spectral libraries
- name: OptimusViewer
  role: opens the Optimus .db file showing metabolite annotations alongside feature
    m/z, RT, and charge for inspection
  repo: https://github.com/alexandrovteam/OptimusViewer
- name: Sirius
  role: third-party in-silico prediction tool for downstream analysis of exported
    spectral data when annotations need follow-up
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nprot.2017.122
    title: 3D molecular cartography using LC–MS facilitated by Optimus and 'ili software
  source_content_hash: sha256:e08c0578cd20370620716c67056610c18f71e83c3356d4746f0849f28a939b3c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MolecularCartography/Optimus
  tools:
  - GNPS
  - KNIME
  - Optimus
  - OptimusViewer
  - Python
  - Sirius
  - pyopenms
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mzrt-molecular-annotation

## Summary

Annotate aligned LC-MS features by matching their m/z and retention time against a list of molecules of interest, yielding putative annotations at Metabolomics Standards Initiative level 2 within the Optimus/KNIME workflow.

## When to use

Use when an aligned LC-MS feature quantification matrix exists and detected consensus features must be putatively identified by mz-RT matching against a molecule list (e.g., GNPS MS/MS matches or a manually built CSV).

## When NOT to use

- You already have validated MS/MS spectral identifications and only need quantification/spatial mapping — annotation at MSI level 2 adds no confidence beyond what you have.
- Input is raw vendor/mzML runs with no aligned feature table; run OpenMS feature detection and alignment in Optimus first.
- Direct-infusion data without retention-time information, where RT matching is not meaningful.

## Inputs

- Aligned feature quantification matrix from Optimus (consensus features with numeric ID, m/z, RT, charge)
- Molecules-of-interest list as CSV (manually created or exported from GNPS MS/MS library matching)
- Optimus .db feature database consumable by OptimusViewer for checking annotations

## Outputs

- Feature quantification matrix with metabolite annotations assigned per consensus feature
- ili_input.csv with annotated features for spatial mapping in 'ili
- Putative (MSI level 2) compound identifications per feature

## How to apply

After OpenMS feature detection and alignment inside the Optimus KNIME workflow, match each consensus feature's m/z and retention time to a molecules-of-interest list. The list can be exported directly from GNPS as a result of MS/MS spectral-library matching, or provided as a manually created CSV; for the phone-swab MSV000079825 reproduction it contains minoxidil (m/z 210.136) and trehalose 6-tripalmitate (m/z 603.335). Apply m/z tolerance of 20 ppm for QTOF-type data and RT tolerance of ~30 s for a 10-min UHPLC gradient. The result is a putative annotation at 'putatively annotated compounds', level 2 of the Metabolomics Standards Initiative (Sumner et al. 2007); MS/MS validation of the annotation is not performed inside Optimus and must be done separately. Annotations propagate into the quantification matrix and the ili_input.csv used for spatial mapping in 'ili.

## Related tools

- **Optimus** (KNIME workflow performing putative molecular annotation of detected features by mz-RT matching to a list of molecules of interest) — https://github.com/MolecularCartography/Optimus
- **GNPS** (source of the molecules-of-interest list exported from MS/MS matching against GNPS spectral libraries)
- **OptimusViewer** (opens the Optimus .db file showing metabolite annotations alongside feature m/z, RT, and charge for inspection) — https://github.com/alexandrovteam/OptimusViewer
- **Sirius** (third-party in-silico prediction tool for downstream analysis of exported spectral data when annotations need follow-up)

## Examples

```
python -c "import pandas as pd; feats=pd.read_csv('features_quantification_matrix.csv'); mols=pd.read_csv('molecules_of_interest.csv'); tol_ppm=20; tol_rt=30; feats['annotation']=[mols.loc[(abs(mols.mz-f.mz)/f.mz*1e6<tol_ppm)&(abs(mols.rt-f.rt)<tol_rt),'name'].tolist() for f in feats.itertuples()]
```

## Evaluation signals

- Annotated features match the molecule list within the specified tolerances: m/z within 20 ppm (QTOF data) and RT within ~30 s for a 10-min UHPLC gradient.
- In the phone-swab reproduction, minoxidil (m/z 210.136) annotates features localized in rows C and G of the 96-well plate and trehalose 6-tripalmitate (m/z 603.335) in row H.
- Blank row A of the 96-well plate contains no annotated features for the molecules of interest.
- Annotations appear in the correct .db column of OptimusViewer (first five columns include metabolite annotations) and in ili_input.csv for mapped features.
- No annotation is claimed above MSI level 2 without separate MS/MS validation.

## Limitations

- This implements identification only at the level of putatively annotated compounds (MSI level 2); MS/MS validation of putative annotations is needed and currently not provided in Optimus.
- Annotations rely entirely on the supplied molecules-of-interest list; features not on the list remain unannotated.
- Tolerances are data-dependent — 20 ppm and ~30 s RT suit QTOF data on a 10-min UHPLC gradient; other instruments or gradients require adjusted windows to avoid false matches.

## Evidence

- [readme] GNPS-sourced molecule list: [evidence span withheld — anchor status: not_found]
- [results] task workflow tolerances: [evidence span withheld — anchor status: not_found]
- [readme] OptimusViewer annotation columns: [evidence span withheld — anchor status: external_doc]
