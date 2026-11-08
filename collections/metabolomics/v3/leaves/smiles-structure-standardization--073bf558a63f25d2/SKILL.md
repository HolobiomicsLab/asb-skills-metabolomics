---
name: smiles-structure-standardization
description: Use when a compound library containing raw SMILES strings and retention
  times must be standardized and made rCDK/ChemAxon-compatible before computing molecular
  descriptors for retention time prediction.
when_to_use_negative:
- The input is already a complete-case, descriptor-ready feature table with 286 rCDK
  descriptor columns — go straight to model training.
- Structures have already been salt-stripped, coordinate-generated, and hydrogenated
  in a prior pass; re-running the standardizer may discard valid compounds.
- The target compounds (e.g. triacylglycerols, phosphatidylcholines) lie outside the
  training chemical landscape and would likely fail rCDK descriptor calculation anyway.
edam_topics:
- http://edamontology.org/topic_2250
- http://edamontology.org/topic_0089
tools:
- name: ChemAxon
  role: Standardizer used to remove salts and metal-containing compounds; cxsmiles
    export with 2D coordinate generation
- name: Retip
  role: Provides the getCD() function that orchestrates descriptor computation and
    removal of failing compounds
  repo: https://github.com/olobion/Retip
- name: rcdk
  role: R CDK wrapper (v3.4.7.1 in original workflow; 3.8.1 in Retip 2.0 install list)
    that computes the 286 chemical descriptors per compound
- name: OpenBabel
  role: Toolkit used to identify/reformat SMILES incompatible with rCDK and ChemAxon
    during curation
- name: alvaDesc
  role: Optional visual inspection of molecular weight histograms, logP distributions,
    and PCA of the descriptor space
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:2f732de06a39b1d26f89b4ee2d0d6fc0dd3ecf4af16f694c09eff294c4b64d99
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - ChemAxon
  - OpenBabel
  - R
  - Retip
  - alvaDesc
  - rcdk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2250
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# smiles-structure-standardization

## Summary

Curates raw compound SMILES from a retention-time training library (e.g. the Retip HILIC dataset) using the ChemAxon standardizer, cxsmiles conversion, 2D coordinate generation, and explicit-hydrogen addition so the rCDK getCD() pipeline can compute 286 chemical descriptors per compound.

## When to use

Use when a compound library containing raw SMILES strings and retention times must be standardized and made rCDK/ChemAxon-compatible before computing molecular descriptors for retention time prediction.

## When NOT to use

- The input is already a complete-case, descriptor-ready feature table with 286 rCDK descriptor columns — go straight to model training.
- Structures have already been salt-stripped, coordinate-generated, and hydrogenated in a prior pass; re-running the standardizer may discard valid compounds.
- The target compounds (e.g. triacylglycerols, phosphatidylcholines) lie outside the training chemical landscape and would likely fail rCDK descriptor calculation anyway.

## Inputs

- Compound library with SMILES strings and measured retention times (e.g. Retip HILIC dataset CSV)
- Raw/unstandardized SMILES strings for each compound
- Optional: descriptor-space inspection inputs for alvaDesc (standardized structure file)

## Outputs

- Standardized .cxsmiles file with atom properties and 2D coordinates and explicit hydrogens
- descriptors_table.csv — complete-case descriptor table with exactly 286 descriptor columns per retained compound
- Set of compounds removed because they failed descriptor calculation

## How to apply

Load the compound library (e.g. the Retip HILIC dataset of SMILES codes and retention times) and curate structures with the ChemAxon standardizer to remove salts and metal-containing compounds, excluding or reformatting SMILES incompatible with rCDK and ChemAxon/OpenBabel. Convert standardized structures to ChemAxon Extended SMILES (.cxsmiles) format storing atom properties and coordinates, then generate 2D coordinates from connectivity data — 2D-based descriptors are deliberately used to avoid 3D computational overhead. Add explicit hydrogens to obtain correct representations and accurate AlogP/XlogP calculations. Finally run the Retip getCD() function, which relies on the rcdk package (version 3.4.7.1 in the original workflow), to compute 286 descriptors per compound while automatically removing compounds that fail descriptor calculation, and export the complete-case table to descriptors_table.csv.

## Related tools

- **ChemAxon** (Standardizer used to remove salts and metal-containing compounds; cxsmiles export with 2D coordinate generation)
- **Retip** (Provides the getCD() function that orchestrates descriptor computation and removal of failing compounds) — https://github.com/olobion/Retip
- **rcdk** (R CDK wrapper (v3.4.7.1 in original workflow; 3.8.1 in Retip 2.0 install list) that computes the 286 chemical descriptors per compound)
- **OpenBabel** (Toolkit used to identify/reformat SMILES incompatible with rCDK and ChemAxon during curation)
- **alvaDesc** (Optional visual inspection of molecular weight histograms, logP distributions, and PCA of the descriptor space)

## Examples

```
library(Retip); desc <- getCD(compounds); stopifnot(ncol(desc) == 286); write.csv(desc, 'descriptors_table.csv')
```

## Evaluation signals

- Verify descriptors_table.csv contains exactly 286 descriptor columns per retained compound.
- Confirm compounds that failed descriptor calculation have been removed — the exported table must be complete-case (no rows with missing descriptors).
- Check that AlogP/XlogP values are plausible, which depends on explicit hydrogens having been added before getCD().
- Use alvaDesc to inspect molecular weight histograms, logP distributions, and PCA of the descriptor space for outliers caused by bad standardization.
- Cross-check that the retained SMILES set no longer includes salts or metal-containing compounds that were flagged during ChemAxon standardization.

## Limitations

- 2D-based molecular descriptors do not fully explain complex LC interactions such as buffer ionic strengths and adsorption/desorption processes, capping retention prediction accuracy near ±1 min MAE.
- Compounds incompatible with rCDK and ChemAxon/OpenBabel are automatically dropped, so the descriptor table is biased toward well-behaved chemical space.
- Retip cannot be installed in a conda environment because rJava requires NVIDIA drivers (Retip 2.0 README), complicating the R setup for this workflow.
- Predictions made outside the chemical landscape of the standardized training set may be more erroneous.

## Evidence

- [methods] Retip's descriptor workflow standardizes structures via ChemAxon, generates 2D coordinates, converts SMILES to cxsmiles format, adds explicit hydrogens: [evidence span withheld — anchor status: not_found]
- [methods] add explicit hydrogens for correct representations and accurate Alog/XlogP calculations: [evidence span withheld — anchor status: not_found]
- [methods] Run the Retip getCD() function, which relies on the rcdk package (version 3.4.7.1), to compute 286 chemical descriptors per compound: [evidence span withheld — anchor status: not_found]
