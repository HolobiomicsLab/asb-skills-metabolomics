---
name: cxsmiles-conversion-and-explicit-hydrogen-addition
description: Use when a compound library of SMILES plus retention times must be prepared
  as descriptor input for Retip's RT prediction models, i.e., before running getCD()
  on rcdk-compatible structures.
when_to_use_negative:
- The input is already a curated descriptor table (e.g., descriptors_table.csv with
  286 columns) — proceed directly to model training instead.
- Structures still contain salts or metal-containing compounds — run ChemAxon standardization
  first, since unstandardized SMILES are incompatible with rcdk and ChemAxon/OpenBabel.
- The analysis requires 3D conformational descriptors — this workflow deliberately
  uses 2D coordinates to avoid 3D computational overhead.
edam_topics:
- http://edamontology.org/topic_2251
tools:
- name: ChemAxon standardizer
  role: Removes salts and metal-containing compounds and reformats SMILES into the
    ChemAxon Extended SMILES (cxsmiles) format storing atom properties and coordinates
- name: rcdk
  role: R wrapper used by Retip's getCD() to compute 286 chemical descriptors from
    the prepared 2D structures (version 3.4.7.1 in the paper; 3.8.1 in Retip 2.0 install
    instructions)
- name: OpenBabel
  role: Structure toolkit compatibility reference for excluding or reformatting SMILES
    that fail in ChemAxon/OpenBabel handling
- name: Retip
  role: R package whose getCD() function wraps descriptor calculation and automatically
    removes compounds that fail descriptor computation
  repo: https://github.com/olobion/Retip
- name: alvaDesc
  role: Optional visual inspection of molecular-weight histograms, logP distributions,
    and PCA of the descriptor space after computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:debabb651da766291fbd46c4c6183092eb03a077b2171826dbcee03bb98b01dc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - ChemAxon
  - ChemAxon standardizer
  - OpenBabel
  - R
  - Retip
  - alvaDesc
  - rcdk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2251
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cxsmiles-conversion-and-explicit-hydrogen-addition

## Summary

A structure-preparation step from the Retip retention-time-prediction pipeline in which standardized SMILES are converted to ChemAxon Extended SMILES (cxsmiles) with 2D coordinates, and explicit hydrogens are added, so that the rcdk-based getCD() function can compute 286 2D chemical descriptors per compound.

## When to use

Use when a compound library of SMILES plus retention times must be prepared as descriptor input for Retip's RT prediction models, i.e., before running getCD() on rcdk-compatible structures.

## When NOT to use

- The input is already a curated descriptor table (e.g., descriptors_table.csv with 286 columns) — proceed directly to model training instead.
- Structures still contain salts or metal-containing compounds — run ChemAxon standardization first, since unstandardized SMILES are incompatible with rcdk and ChemAxon/OpenBabel.
- The analysis requires 3D conformational descriptors — this workflow deliberately uses 2D coordinates to avoid 3D computational overhead.

## Inputs

- Compound library (e.g., the Retip HILIC dataset of 981 primary metabolites and biogenic amines) with SMILES codes and retention times
- SMILES strings standardized via ChemAxon (salts and metal-containing compounds removed)
- ChemAxon Extended SMILES (*.cxsmiles) files storing atom properties and coordinates

## Outputs

- cxsmiles-format structures with 2D coordinates generated from connectivity data and explicit hydrogens added
- Cleaned, complete-case descriptor table exported as descriptors_table.csv containing exactly 286 descriptor columns per retained compound

## How to apply

First curate structures with the ChemAxon standardizer to remove salts and metal-containing compounds whose SMILES are incompatible with rcdk and ChemAxon/OpenBabel. Convert the standardized structures to ChemAxon Extended SMILES (*.cxsmiles) format that stores atom properties and coordinates, then generate 2D coordinates from the connectivity data — 2D-based descriptors are used deliberately to avoid the computational overhead of 3D descriptor generation. Add explicit hydrogens to each structure because correct hydrogen representations are required for accurate Alog and XlogP calculations. Pass the prepared structures to Retip's getCD() function (built on rcdk version 3.4.7.1), which computes 286 chemical descriptors per compound and automatically removes compounds that fail descriptor calculation. Optionally inspect the resulting descriptor space (molecular-weight histograms, logP distributions, PCA) with alvaDesc before exporting descriptors_table.csv.

## Related tools

- **ChemAxon standardizer** (Removes salts and metal-containing compounds and reformats SMILES into the ChemAxon Extended SMILES (cxsmiles) format storing atom properties and coordinates)
- **rcdk** (R wrapper used by Retip's getCD() to compute 286 chemical descriptors from the prepared 2D structures (version 3.4.7.1 in the paper; 3.8.1 in Retip 2.0 install instructions))
- **OpenBabel** (Structure toolkit compatibility reference for excluding or reformatting SMILES that fail in ChemAxon/OpenBabel handling)
- **Retip** (R package whose getCD() function wraps descriptor calculation and automatically removes compounds that fail descriptor computation) — https://github.com/olobion/Retip
- **alvaDesc** (Optional visual inspection of molecular-weight histograms, logP distributions, and PCA of the descriptor space after computation)

## Evaluation signals

- The exported descriptors_table.csv contains exactly 286 descriptor columns per retained compound.
- Compounds that failed descriptor calculation have been removed, leaving a complete-case table.
- Explicit hydrogens are present in the prepared structures, confirming Alog and XlogP can be computed accurately.
- cxsmiles output preserves atom properties and 2D coordinates generated from the connectivity data.
- alvaDesc inspections (MW histograms, logP distributions, PCA of descriptor space) show a chemically coherent descriptor distribution.

## Limitations

- Compounds whose SMILES are incompatible with rcdk, ChemAxon, or OpenBabel are excluded or must be reformatted, and compounds failing descriptor calculation are automatically dropped.
- 2D-based descriptors cannot fully explain complex LC interactions such as buffer systems (including ionic strengths), microenvironments, and adsorption/desorption processes, which caps model accuracy at about ±1 min MAE.
- Molecular descriptors alone may be insufficiently complex for retention prediction, and predictions outside the chemical landscape of the training set may be more erroneous.

## Evidence

- [methods] cxsmiles conversion and 2D coordinates: [evidence span withheld — anchor status: not_found]
- [methods] explicit hydrogens rationale: [evidence span withheld — anchor status: not_found]
- [methods] getCD descriptor computation: [evidence span withheld — anchor status: not_found]
- [methods] standardization prerequisite: [evidence span withheld — anchor status: not_found]
