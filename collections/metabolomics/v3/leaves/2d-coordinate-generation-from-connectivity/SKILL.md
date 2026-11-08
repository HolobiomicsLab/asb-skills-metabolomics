---
name: 2d-coordinate-generation-from-connectivity
description: Use when a compound library holds SMILES strings without coordinates
  and 2D-based chemical descriptors (e.g. AlogP/XlogP) must be computed for retention
  time prediction; 3D coordinate generation is unnecessary or too costly.
when_to_use_negative:
- When 3D conformational descriptors are required — the Retip workflow deliberately
  uses 2D-based descriptors to avoid 3D computational overhead, so this skill cannot
  supply 3D information
- When the input SMILES are unstandardized (contain salts or metal-containing species)
  or are incompatible with rCDK and ChemAxon/OpenBabel — they must be curated first
  or compounds will fail descriptor calculation
- When the goal is retention prediction for chemistry far outside the training set's
  chemical landscape (e.g. complex high-molecular-weight lipids like triacylglycerols
  or phosphatidylcholines, which were largely excluded in this study)
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3318
tools:
- name: rcdk
  role: R CDK wrapper used by Retip's getCD() to compute the 286 descriptors from
    the coordinate-bearing structures (version 3.4.7.1 in the paper; 3.8.1 in Retip
    2.0)
- name: ChemAxon standardizer
  role: Upstream structure standardization and cxsmiles conversion that provides the
    cleaned connectivity from which 2D coordinates are generated
- name: Retip
  role: Hosts the getCD() descriptor pipeline that consumes the 2D-coordinate structures
    and emits the 286-descriptor table
  repo: https://github.com/olobion/Retip
- name: alvaDesc
  role: Optional visual inspection of molecular weight histograms, logP distributions,
    and PCA of the descriptor space after coordinate-based computation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:b12d8017c74c749880ef4c20b64f8d67a86f19bacb8f5bb1b95ca56dc98dda0d
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
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# 2D coordinate generation from connectivity

## Summary

Generates 2D atomic coordinates from molecular connectivity (SMILES) as part of the Retip descriptor-computation pipeline, producing cxsmiles representations with atom properties and explicit hydrogens that feed rCDK's 286-descriptor calculation for retention time prediction.

## When to use

Use when a compound library holds SMILES strings without coordinates and 2D-based chemical descriptors (e.g. AlogP/XlogP) must be computed for retention time prediction; 3D coordinate generation is unnecessary or too costly.

## When NOT to use

- When 3D conformational descriptors are required — the Retip workflow deliberately uses 2D-based descriptors to avoid 3D computational overhead, so this skill cannot supply 3D information
- When the input SMILES are unstandardized (contain salts or metal-containing species) or are incompatible with rCDK and ChemAxon/OpenBabel — they must be curated first or compounds will fail descriptor calculation
- When the goal is retention prediction for chemistry far outside the training set's chemical landscape (e.g. complex high-molecular-weight lipids like triacylglycerols or phosphatidylcholines, which were largely excluded in this study)

## Inputs

- ChemAxon-standardized compound table with SMILES codes and retention times (e.g. the Retip HILIC dataset of 981 primary metabolites and biogenic amines)
- SMILES connectivity strings curated to be compatible with rCDK and ChemAxon/OpenBabel toolkits
- cxsmiles-format structure records

## Outputs

- cxsmiles records with generated 2D coordinates, atom properties, and explicit hydrogens
- Descriptor table (descriptors_table.csv) with 286 descriptor columns per retained compound
- Record of compounds removed for failed descriptor calculation

## How to apply

Start from the ChemAxon-standardized structure table (salts and metal-containing compounds removed; incompatible SMILES excluded or reformatted) and convert structures to ChemAxon Extended SMILES (*.cxsmiles) format that stores atom properties and coordinates. Generate 2D coordinates from the connectivity data — the Retip workflow deliberately uses 2D-based descriptors to avoid 3D computational overhead — and add explicit hydrogens so representations are correct and Alog/XlogP-type partition-coefficient descriptors are accurate. Feed the coordinate-bearing structures into Retip's getCD() function, which relies on the rcdk package (version 3.4.7.1 in the published workflow; 3.8.1 in Retip 2.0's dependency list) to compute 286 descriptors per compound. Compounds that fail descriptor calculation are automatically removed. Optionally inspect the resulting descriptor space (molecular weight histograms, logP distributions, PCA) with alvaDesc before exporting descriptors_table.csv.

## Related tools

- **rcdk** (R CDK wrapper used by Retip's getCD() to compute the 286 descriptors from the coordinate-bearing structures (version 3.4.7.1 in the paper; 3.8.1 in Retip 2.0))
- **ChemAxon standardizer** (Upstream structure standardization and cxsmiles conversion that provides the cleaned connectivity from which 2D coordinates are generated)
- **Retip** (Hosts the getCD() descriptor pipeline that consumes the 2D-coordinate structures and emits the 286-descriptor table) — https://github.com/olobion/Retip
- **alvaDesc** (Optional visual inspection of molecular weight histograms, logP distributions, and PCA of the descriptor space after coordinate-based computation)

## Examples

```
install.packages('rcdk', version='3.8.1', repos='http://cran.rstudio.com/'); devtools::install_github('olobion/Retip'); descriptors_table <- getCD(smiles_table)
```

## Evaluation signals

- The exported descriptors_table.csv contains exactly 286 descriptor columns per retained compound
- Compounds that failed descriptor calculation have been removed from the table (complete-case descriptor table)
- Explicit hydrogens are present in the processed structures, since these are required for accurate Alog/XlogP calculations
- cxsmiles records carry stored atom properties and 2D coordinates derived from the connectivity data
- Spot-check descriptor distributions (e.g. logP, molecular weight histograms via alvaDesc) for anomalies indicating failed coordinate or hydrogen addition

## Limitations

- Compounds that fail descriptor calculation are automatically removed, so the output table can shrink relative to the input library
- 2D-based descriptors were chosen to avoid 3D computational overhead; the paper notes molecular descriptors alone may fail to fully explain complex interactions between LC buffer systems, microenvironments, and adsorption/desorption processes
- Descriptor quality depends on upstream standardization — SMILES incompatible with rCDK and ChemAxon/OpenBabel must be excluded or reformatted first

## Evidence

- [methods] Generate 2D coordinates from connectivity data: [evidence span withheld — anchor status: not_found]
- [methods] cxsmiles format stores coordinates: [evidence span withheld — anchor status: not_found]
- [methods] getCD computes 286 descriptors: [evidence span withheld — anchor status: not_found]
- [readme] rcdk dependency pinned in README: "install.packages('rcdk', version='3.8.1', repos='http://cran.rstudio.com/')
install.packages('rcdklibs', version='2.9', repos='http://cran.rstudio.com/')"
