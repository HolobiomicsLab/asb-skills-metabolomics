---
name: small-molecule-structure-descriptor-calculation
description: Use when you have SMILES (or equivalent 2D structure) representations
  for small-molecule databases like HMDB, DrugBank, FooDB, or STOFF and need quantitative
  molecular descriptors to feed an RT-prediction or QSAR-style machine learning model.
when_to_use_negative:
- Compounds fall outside the training chemical space — complex, high molecular weight
  lipids such as triacylglycerols or phosphatidylcholines were largely excluded, so
  descriptors for these will drive out-of-domain predictions
- Structures cannot be parsed into valid 2D representations (e.g. missing or malformed
  SMILES), since CDK descriptor computation will fail or yield unreliable values
- You need chromatographic-method-specific features rather than structure-intrinsic
  descriptors — molecular descriptors may fail to fully explain LC buffer, ionic-strength,
  and adsorption/desorption interactions
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: Retip
  role: Provides the getCD() function for computing 286 CDK descriptors from SMILES
    and cesc() for centering/scaling
  repo: https://github.com/olobion/Retip
- name: Retiplib
  role: R package bundling SMILES collections from the 18+ public biochemical databases
    used for descriptor computation
  repo: https://github.com/olobion/Retiplib
- name: rcdk
  role: R interface to the Java Chemistry Development Kit that executes the descriptor
    calculations (requires rJava and Java JDK)
- name: Keras
  role: Downstream consumer of the scaled descriptor matrix for retention time prediction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:bf8865d1d5ce70594e9866d53faa48ef8277b62ae554c3026f64dcbc638022c0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - Keras
  - R
  - Retip
  - Retiplib
  - rcdk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# Small-molecule structure descriptor calculation

## Summary

Computes 286 CDK molecular descriptors from SMILES structures (via the Retip getCD() function built on the rcdk/rcdklibs Java-based Chemistry Development Kit) to featurize small molecules for retention-time prediction models in untargeted metabolomics.

## When to use

Use when you have SMILES (or equivalent 2D structure) representations for small-molecule databases like HMDB, DrugBank, FooDB, or STOFF and need quantitative molecular descriptors to feed an RT-prediction or QSAR-style machine learning model.

## When NOT to use

- Compounds fall outside the training chemical space — complex, high molecular weight lipids such as triacylglycerols or phosphatidylcholines were largely excluded, so descriptors for these will drive out-of-domain predictions
- Structures cannot be parsed into valid 2D representations (e.g. missing or malformed SMILES), since CDK descriptor computation will fail or yield unreliable values
- You need chromatographic-method-specific features rather than structure-intrinsic descriptors — molecular descriptors may fail to fully explain LC buffer, ionic-strength, and adsorption/desorption interactions

## Inputs

- SMILES strings (or equivalent 2D structure representations) for each small-molecule compound
- Compound identifier-to-SMILES mapping tables from public biochemical databases (HMDB, DrugBank, FooDB, STOFF, etc., bundled in the Retiplib package)

## Outputs

- A numeric descriptor matrix with 286 CDK descriptors per compound (one row per compound)
- Center-and-scale transformed descriptor matrix (via cesc()) ready for machine-learning input

## How to apply

Install Retip and Retiplib from GitHub (devtools::install_github('olobion/Retip')) alongside rcdk 3.8.1, rcdklibs 2.9, rJava, and Java JDK, since descriptor calculation runs through the Java CDK via rJava. Load SMILES structures for each compound (Retiplib bundles databases such as HMDB, KNApSAcK, ChEBI, DrugBank, SMPDB, YMDB, T3DB, FooDB, NANPDB, STOFF, BMDB, LipidMAPS, Urine, Saliva, Feces, ECMDB, CSF, Serum, PubChem.1, PlantCyc, UNPD, BLEXP, NPA and COCONUT) and compute the 286 CDK descriptors per structure with the Retip getCD() function. Center and scale the resulting descriptor matrix with the cesc() function before feeding it to Keras or other ML models, as neural-network predictions require standardized inputs. Note that the octanol/water partition coefficient is among the most informative descriptors for both RPLC and HILIC retention behavior.

## Related tools

- **Retip** (Provides the getCD() function for computing 286 CDK descriptors from SMILES and cesc() for centering/scaling) — https://github.com/olobion/Retip
- **Retiplib** (R package bundling SMILES collections from the 18+ public biochemical databases used for descriptor computation) — https://github.com/olobion/Retiplib
- **rcdk** (R interface to the Java Chemistry Development Kit that executes the descriptor calculations (requires rJava and Java JDK))
- **Keras** (Downstream consumer of the scaled descriptor matrix for retention time prediction)

## Examples

```
install.packages('rcdk', version='3.8.1', repos='http://cran.rstudio.com/'); install.packages('rcdklibs', version='2.9', repos='http://cran.rstudio.com/'); devtools::install_github('olobion/Retiplib'); devtools::install_github('olobion/Retip')
```

## Evaluation signals

- The output descriptor table contains exactly one row per input compound and 286 CDK descriptor columns per structure
- Descriptors are numeric and finite for every structure — unparseable SMILES or missing CDK values indicate failed Java/rJava setup
- Before model fitting, the descriptor matrix must be centered and scaled (cesc()); downstream Keras RT predictions should be non-negative
- Spot-checked compounds' descriptor-derived RT predictions agree with MS-FINDER 3.24+ shipped RT values within the reported ±1 min MAE tolerance (0.78 min HILIC, 0.57 min RPLC)

## Limitations

- Molecular descriptors alone do not fully explain complex interactions between LC buffer systems (including ionic strengths), microenvironments, and adsorption/desorption processes
- Prediction accuracy is generally limited by the number of compounds used for training and their chemical diversity; chemicals poorly represented by the training models yield more erroneous predictions
- Retip installation is not possible in conda environments because rJava requires specific Java setup (README notes NVIDIA driver issues in conda)

## Evidence
