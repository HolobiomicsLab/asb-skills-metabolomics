---
name: cdk-chemical-descriptor-calculation
description: Use when a compound library (SMILES + experimental retention times, e.g.
  HILIC MoNA or RPLC RIKEN PlaSMA sets) must be converted into a numeric molecular-descriptor
  feature table for training or applying retention time prediction models.
when_to_use_negative:
- Input is already a computed descriptor table — proceed directly to model fitting
  (caret split, cesc() centering/scaling, proc.data() cleaning)
- The task requires 3D conformer-dependent descriptors — Retip's pipeline deliberately
  uses 2D-based descriptors to avoid 3D computational overhead
- Compounds contain salts or metals that will fail rCDK — standardize or exclude them
  first rather than passing raw SMILES to getCD()
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Retip
  role: Provides the getCD() function that orchestrates CDK descriptor computation
    and drops compounds that fail calculation
  repo: https://github.com/olobion/Retip
- name: rcdk
  role: rJava-backed Chemistry Development Kit binding that actually computes the
    286 descriptors (rcdk 3.4.7.1 originally; 3.8.1 in Retip 2.0)
- name: ChemAxon
  role: Standardizer used upstream to remove salts/metal compounds and generate cxsmiles
    with 2D coordinates and explicit hydrogens
- name: alvaDesc
  role: Optional visual inspection of descriptor space (MW histograms, logP distributions,
    PCA)
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:a7cceba3fdd30a5a3f610bb8a7979e823b1bfb47caa1d15b9cc4b6125b01bb6f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - ChemAxon
  - LightGBM
  - OpenBabel
  - R
  - Retip
  - alvaDesc
  - caret
  - rcdk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cdk-chemical-descriptor-calculation

## Summary

Computes 286 CDK molecular descriptors per compound from standardized SMILES structures using Retip's getCD() function built on the rcdk package, producing the numeric feature matrix that feeds Retip's retention-time prediction models (RF, BRNN, XGBoost, LightGBM, Keras, H2O autoML).

## When to use

Use when a compound library (SMILES + experimental retention times, e.g. HILIC MoNA or RPLC RIKEN PlaSMA sets) must be converted into a numeric molecular-descriptor feature table for training or applying retention time prediction models.

## When NOT to use

- Input is already a computed descriptor table — proceed directly to model fitting (caret split, cesc() centering/scaling, proc.data() cleaning)
- The task requires 3D conformer-dependent descriptors — Retip's pipeline deliberately uses 2D-based descriptors to avoid 3D computational overhead
- Compounds contain salts or metals that will fail rCDK — standardize or exclude them first rather than passing raw SMILES to getCD()

## Inputs

- Compound library with SMILES strings and experimental retention times (e.g. HILIC set: 880 MoNA compounds; RPLC set: 398 RIKEN PlaSMA compounds)
- ChemAxon-standardized structures in cxsmiles format with 2D coordinates and explicit hydrogens

## Outputs

- descriptors_table.csv — complete-case table with exactly 286 CDK descriptor columns per retained compound
- Optional alvaDesc QC artifacts (molecular weight histograms, logP distributions, PCA plots of descriptor space)

## How to apply

Load the compound library with SMILES codes and retention times, then curate structures with the ChemAxon standardizer to remove salts and metal-containing compounds incompatible with rCDK. Convert standardized structures to ChemAxon Extended SMILES (.cxsmiles) storing atom properties, generate 2D coordinates from connectivity data (2D-based descriptors are used deliberately to avoid 3D computational overhead), and add explicit hydrogens so hydrophobicity descriptors such as Alog/XlogP are calculated accurately. Run the Retip getCD() function, which relies on rcdk (version 3.4.7.1 in the original paper, 3.8.1 in Retip 2.0), to compute 286 descriptors per compound; compounds that fail descriptor calculation are automatically removed. Optionally inspect the descriptor space with alvaDesc (molecular weight histograms, logP distributions, PCA). Export the cleaned, complete-case table to descriptors_table.csv and verify it contains exactly 286 descriptor columns per retained compound.

## Related tools

- **Retip** (Provides the getCD() function that orchestrates CDK descriptor computation and drops compounds that fail calculation) — https://github.com/olobion/Retip
- **rcdk** (rJava-backed Chemistry Development Kit binding that actually computes the 286 descriptors (rcdk 3.4.7.1 originally; 3.8.1 in Retip 2.0))
- **ChemAxon** (Standardizer used upstream to remove salts/metal compounds and generate cxsmiles with 2D coordinates and explicit hydrogens)
- **alvaDesc** (Optional visual inspection of descriptor space (MW histograms, logP distributions, PCA))

## Examples

```
library(Retip); descriptors_table <- getCD(compounds)  # 286 CDK descriptors via rcdk; failing compounds removed
```

## Evaluation signals

- Output table contains exactly 286 descriptor columns per retained compound
- Compounds that failed descriptor calculation have been removed (complete-case table; row count matches compounds surviving standardization)
- XlogP values are plausible — explicit hydrogens were added, since missing H atoms distort Alog/XlogP calculations
- Descriptor table is compatible with downstream proc.data(): no non-existent values and low-variance columns can be cleaned before model training
- Retip 2.0 environment check: rJava loads under R 4.4.0 with Java JDK configured (R CMD javareconf)

## Limitations

- Compounds that fail rCDK descriptor calculation are silently dropped, so the descriptor table is a complete-case subset of the input library
- The article notes molecular descriptors 'seem to fail to fully explain the complex interactions between LC buffer systems (including ionic strengths), microenvironments' — none of the current models reach validation MAEs better than ±1 min
- Descriptors computed for chemicals outside the training-set chemical landscape may be less reliable; complex high-molecular-weight lipids (triacylglycerols, phosphatidylcholines) were excluded from training
- Retip cannot be installed in conda environments because rJava requires NVIDIA drivers; Java JDK/rJava setup is a common failure point

## Evidence

- [abstract] A complete workflow for retention time prediction was developed in R.: "A complete workflow for retention time prediction was developed in R."
- [discussion] molecular descriptors by themselves seem to fail to fully explain the complex interactions between LC buffer systems (including ionic strengths), microenvironments (such as static aqueous phases in: "molecular descriptors by themselves seem to fail to fully explain the complex interactions between LC buffer systems (including ionic strengths), microenvironments (such as static aqueous phases in"
- [readme] install.packages('rcdk', version='3.8.1', repos='http://cran.rstudio.com/'): "install.packages('rcdk', version='3.8.1', repos='http://cran.rstudio.com/')"
- [readme] Retention time calculation can be useful in identifying unknowns and removing false positive annotations.: "Retention time calculation can be useful in identifying unknowns and removing false positive annotations."
