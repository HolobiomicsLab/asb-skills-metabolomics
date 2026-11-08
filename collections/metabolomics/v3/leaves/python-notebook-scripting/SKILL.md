---
name: python-notebook-scripting
description: Use when the analysis must be delivered as an interactive, user-editable
  Python notebook (.ipynb) that combines machine learning prediction, descriptor calculation,
  and filtering in one runnable artifact.
when_to_use_negative:
- If the workflow requires prediction of adduct types beyond [M-H]-, [M+H]+, [M+Na]+,
  or [M+K]+ — the current training sets do not support dimers like [2M+H]+ or [2M-H]-
  or in-source fragments
- If the user needs a fully scripted, headless pipeline rather than an interactive
  notebook — CCSP 2.0 is explicitly delivered as a notebook, not a standalone CLI
  tool
- If CCS prediction must span multiple ion-mobility instrument types (drift tube vs
  traveling wave) — the notebook's training data and validation were tied to the McLean
  CCS Compendium measurements
edam_operation: http://edamontology.org/operation_3662
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CCSP 2.0
  role: The open-source Python notebook being packaged and distributed; provides the
    CCS prediction and annotation-filtering functionality itself
  repo: https://github.com/facundof2016/CCSP2.0
- name: Jupyter Notebook
  role: 'Execution environment: the notebook artifact is run in either Google Colaboratory
    or local Jupyter Lab/Notebook'
  repo: https://jupyter.org/
- name: Mordred
  role: Computes 1613 molecular descriptors from candidate structures within the notebook,
    feeding the linear support vector regression models
- name: Google Colaboratory
  role: Cloud-hosted execution environment for the Colab notebook variant, recommended
    for runs of <10,000 molecules
  repo: https://colab.research.google.com/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:0ef4ec9f91c712d853afbfd8fe3af97136fe411e272a47781b857526751d2fcb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - Biopython
  - CCSP 2.0
  - Google Colaboratory
  - Jupyter Notebook
  - Mordred
  - PubChemPy
  edam_operation: http://edamontology.org/operation_3662
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# python-notebook-scripting

## Summary

Package the CCS prediction and annotation-filtering analysis as a self-contained, open-source Jupyter/Colab notebook so ion-mobility scientists can run user-curated CCS training sets end-to-end without standalone code.

## When to use

Use when the analysis must be delivered as an interactive, user-editable Python notebook (.ipynb) that combines machine learning prediction, descriptor calculation, and filtering in one runnable artifact.

## When NOT to use

- If the workflow requires prediction of adduct types beyond [M-H]-, [M+H]+, [M+Na]+, or [M+K]+ — the current training sets do not support dimers like [2M+H]+ or [2M-H]- or in-source fragments
- If the user needs a fully scripted, headless pipeline rather than an interactive notebook — CCSP 2.0 is explicitly delivered as a notebook, not a standalone CLI tool
- If CCS prediction must span multiple ion-mobility instrument types (drift tube vs traveling wave) — the notebook's training data and validation were tied to the McLean CCS Compendium measurements

## Inputs

- User-curated CCS training set (ion mobility-mass spectrometry measurements with associated compound structures)
- Target compound m/z and adduct type ([M-H]-, [M+H]+, [M+Na]+, [M+K]+)
- Python notebook file (.ipynb) downloaded from the CCSP2.0 repository

## Outputs

- Predicted CCS values with median relative errors of ~1.25–1.87% (adduct-dependent)
- Filtered annotation lists retaining 100% of correct annotations at a ΔCCS threshold of 2.8% and 10 ppm mass error
- Notebook-executed prediction and filtering results within the .ipynb environment

## How to apply

CCSP 2.0 is written in Python and packaged as two notebook variants: a Google Colaboratory .ipynb for beginners (no local Python installation; all computation on Google-hosted services) and a Jupyter Lab-compatible notebook with a Tk interface for local workflows. Download the notebook from the facundof2016/CCSP2.0 GitHub repository, open it in Colab (File > Open Notebook > Upload) or in Jupyter Lab/Notebook, and follow the embedded instructions. Internally the notebook uses Mordred to compute 1613 molecular descriptors, trains linear support vector regression models on user-curated CCS training sets, and applies a ΔCCS threshold of 2.8% with a 10 ppm mass error window to filter false-positive annotations. Small-scale runs (<10,000 molecules) should use the Colab variant; large training/target sets or integration into larger pipelines should use the local Jupyter Lab version.

## Related tools

- **CCSP 2.0** (The open-source Python notebook being packaged and distributed; provides the CCS prediction and annotation-filtering functionality itself) — https://github.com/facundof2016/CCSP2.0
- **Jupyter Notebook** (Execution environment: the notebook artifact is run in either Google Colaboratory or local Jupyter Lab/Notebook) — https://jupyter.org/
- **Mordred** (Computes 1613 molecular descriptors from candidate structures within the notebook, feeding the linear support vector regression models)
- **Google Colaboratory** (Cloud-hosted execution environment for the Colab notebook variant, recommended for runs of <10,000 molecules) — https://colab.research.google.com/

## Examples

```
Download 'CCSP 2.0 - CCS Prediction for Google Colab.ipynb' → upload in Google Colab (File > Open Notebook > Upload) → follow the embedded instructions
```

## Evaluation signals

- The .ipynb file executes without error in either Colab or Jupyter Lab and produces CCS predictions on a user-curated training set
- Predicted CCS values show median relative errors below ~2% when validated against held-out McLean CCS Compendium measurements
- Annotation filtering with the notebook retains 100% of correct annotations at ΔCCS = 2.8% and 10 ppm mass error, while removing 36.1% of incorrect structures in class-matched data sets
- The notebook requires Sklearn V1.0.2 or later; running against an older scikit-learn version should fail or produce inconsistent results
- Output predictions are compared to three existing ML CCS methods (MetCCS, LipidCCS, AllCCS) using 101 randomly selected test sets held consistent across algorithms

## Limitations

- Google Colab will only allow continuous notebook operation for up to 12 hours and disconnects after ~20 minutes if left idle
- The user must define the training set before predicting CCS values — one drawback of most ML methods
- Current training sets do not allow prediction of dimers ([2M+H]+, [2M-H]-) or other less common adducts or in-source fragments
- Colab is recommended only for small-scale predictions (<10,000 molecules); large sets should use the local Jupyter Lab version
- Sklearn V1.0.2 or later is required (see requirements.txt)

## Evidence
