---
name: jupyter-notebook-workflow-execution
description: Use when a computational workflow is distributed as a self-documented
  .ipynb notebook (e.g. CCS prediction, metabolomics filtering) that must be run end-to-end
  in Jupyter Lab, Jupyter Notebook, or Google Colab with all packages installed per
  its requirements file.
when_to_use_negative:
- The task requires running dimers or less common adducts ([2 M + H]+, [2 M − H]−,
  in-source fragments) — CCSP 2.0's current training sets do not support these adduct
  types
- A non-notebook, scriptable CLI or library API is needed — this skill is specific
  to executing .ipynb notebooks with embedded instructions
- Idle sessions matter — Google Colab disconnects after ~20 minutes idle and limits
  continuous operation to 12 hours, so long-running jobs need the local Jupyter Lab
  variant
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: Jupyter Notebook
  role: 'Execution environment: opens and runs the downloaded .ipynb notebook file
    under Python 3'
  repo: https://github.com/facundof2016/CCSP2.0
- name: CCSP 2.0
  role: The notebook being executed — open-source Python notebook for CCS prediction
    and metabolomics false-positive filtering
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Python package invoked within the notebook to compute the 1613 molecular descriptors
    used by the SVR models
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:3d4813f71ffd35da17c772a76070b5de9649db869b96e0ce3401b16199d0b1b7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# jupyter-notebook-workflow-execution

## Summary

Executes scientific Python workflows packaged as Jupyter notebook (.ipynb) files — here, the CCSP 2.0 collision cross section predictor — by loading the notebook in Jupyter Lab/Notebook or Google Colab, running its embedded cells (Mordred descriptor computation, linear SVR CCS prediction), and following the instructions embedded within the notebook.

## When to use

Use when a computational workflow is distributed as a self-documented .ipynb notebook (e.g. CCS prediction, metabolomics filtering) that must be run end-to-end in Jupyter Lab, Jupyter Notebook, or Google Colab with all packages installed per its requirements file.

## When NOT to use

- The task requires running dimers or less common adducts ([2 M + H]+, [2 M − H]−, in-source fragments) — CCSP 2.0's current training sets do not support these adduct types
- A non-notebook, scriptable CLI or library API is needed — this skill is specific to executing .ipynb notebooks with embedded instructions
- Idle sessions matter — Google Colab disconnects after ~20 minutes idle and limits continuous operation to 12 hours, so long-running jobs need the local Jupyter Lab variant

## Inputs

- .ipynb notebook file downloaded from the GitHub repository (e.g. 'CCSP 2.0 - CCS Prediction in Jupyter')
- Python 3 environment (Jupyter Lab, Jupyter Notebook, or Google Colab)
- requirements.txt package list (requires scikit-learn V1.0.2 or later)
- user-curated CCS training set and target molecule structures/measurements (e.g. McLean CCS Compendium values, PubChem candidate lists)

## Outputs

- Executed notebook state with computed CCS predictions from the embedded linear SVR models
- Predicted CCS values and derived ΔCCS filtering statistics for candidate structures (e.g. written to ccs_filtering_results.csv)
- Console/notebook cell outputs from Mordred descriptor computation and SVR fitting

## How to apply

Download the target notebook (e.g. 'CCSP 2.0 - CCS Prediction in Jupyter', saved as a .ipynb file) from its GitHub repository, then choose the environment: for small-scale runs (<10,000 molecules) use the Google Colab variant via File > Open Notebook > Upload, noting Colab's 12-hour session limit and ~20-minute idle disconnect; for large training/target sets or integration into larger workflows, run the locally hosted Jupyter Lab version under Python 3 (e.g. via Anaconda), ensuring scikit-learn V1.0.2 or later is installed per requirements.txt. Execute the embedded cells in order — in the CCSP 2.0 case this computes the 1613 Mordred descriptors, fits linear support vector regression models on user-curated training sets, and predicts CCS values for candidate structures — following the instructions embedded within the notebook rather than improvising steps. Validate outputs against reported benchmarks (e.g. median relative errors of 1.25/1.73/1.87% for [M−H]−/[M+H]+/[M+Na]+ adducts from the McLean CCS Compendium) and export result files such as ccs_filtering_results.csv.

## Related tools

- **Jupyter Notebook** (Execution environment: opens and runs the downloaded .ipynb notebook file under Python 3) — https://github.com/facundof2016/CCSP2.0
- **CCSP 2.0** (The notebook being executed — open-source Python notebook for CCS prediction and metabolomics false-positive filtering) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Python package invoked within the notebook to compute the 1613 molecular descriptors used by the SVR models)

## Examples

```
jupyter lab "CCSP 2.0 - CCS Prediction in Jupyter.ipynb"  # then execute all cells in order
```

## Evaluation signals

- The notebook file opens and executes without import errors in Jupyter Lab/Notebook with scikit-learn ≥1.0.2 installed
- For CCSP 2.0 test validation, median relative CCS prediction errors fall within the reported 1.25/1.73/1.87% for [M−H]−/[M+H]+/[M+Na]+ adducts
- Filtering outputs reproduce reported statistics, e.g. 100% retention of correct annotations and 36.1% filtering of incorrect structures at ΔCCS 2.8% and 10 ppm mass error
- Results are exported to the expected file (e.g. ccs_filtering_results.csv) matching the notebook's embedded instructions

## Limitations

- Requires scikit-learn V1.0.2 or later; a full package list is in requirements.txt
- Google Colab option only supports continuous operation up to 12 hours and disconnects after ~20 minutes if idle
- Current training sets allow prediction of [M − H]−, [M + H]+, [M + Na]+, and [M + K]+ but not dimers such as [2 M + H]+ or [2 M − H]−, or other less common adducts or in-source fragments
- One drawback of most ML methods is the need to define the specific training set prior to predicting CCS values
- Small-scale predictions (<10,000 molecules) are suited to Google Colab; large training/target sets require the locally hosted Jupyter Lab version

## Evidence

- [methods] Embedded instructions as run protocol: [evidence span withheld — anchor status: not_found]
