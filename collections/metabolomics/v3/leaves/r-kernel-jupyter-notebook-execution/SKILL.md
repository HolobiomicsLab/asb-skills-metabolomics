---
name: r-kernel-jupyter-notebook-execution
description: Use when executing the Bioactive_Molecular_Networks Step 2 Jupyter notebook
  that computes Pearson correlation scores/p-values in R, or when Jupyter reports
  no R kernel available.
when_to_use_negative:
- You are still on Step 1 (MZmine2/Optimus LC-MS/MS processing and bioassay table
  editing) or Step 3 (GNPS FBMN job and Cytoscape visualization) — no R kernel is
  needed there.
- The bioassay values per sample have not yet been added to the feature quantification
  table; the Pearson correlation step cannot run on an unannotated table.
- You intend to reproduce the analysis in a standalone R script outside the notebook
  — possible, but it departs from the published reproducible notebook workflow.
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: IRkernel
  role: Native R kernel that lets Jupyter execute the notebook's R correlation code
  repo: https://github.com/IRkernel/IRkernel
- name: Jupyter Notebook
  role: Web application hosting and running the Step 2 bioactivity-scoring notebook
- name: R
  role: Language in which the Pearson correlation and p-value computation are implemented
- name: Anaconda
  role: Convenient environment manager for installing Jupyter and dependency stacks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:0b1a51e6f9b62ad9980e100afca5e90797d32097258c1cd57ea582d7f47b3e4e
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/IRkernel/IRkernel
  tools:
  - Anaconda
  - IRkernel
  - Jupyter Notebook
  - MZmine2
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# r-kernel-jupyter-notebook-execution

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Runs the bioactive molecular networking Step 2 notebook, which executes R code (Pearson correlation of every detected feature against fraction bioassay results) inside Jupyter via the IRkernel, keeping the published workflow reproducible.

## When to use

Use when executing the Bioactive_Molecular_Networks Step 2 Jupyter notebook that computes Pearson correlation scores/p-values in R, or when Jupyter reports no R kernel available.

## When NOT to use

- You are still on Step 1 (MZmine2/Optimus LC-MS/MS processing and bioassay table editing) or Step 3 (GNPS FBMN job and Cytoscape visualization) — no R kernel is needed there.
- The bioassay values per sample have not yet been added to the feature quantification table; the Pearson correlation step cannot run on an unannotated table.
- You intend to reproduce the analysis in a standalone R script outside the notebook — possible, but it departs from the published reproducible notebook workflow.

## Inputs

- features_quantification_matrix_edited_bioactivity.csv (MZmine2-exported feature quantification table with row ID, row m/z, row retention time, peak area, plus per-sample bioassay values)
- Transposed feature-by-fraction quantification matrix
- Per-fraction bioassay result vector

## Outputs

- features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv (per-feature Pearson correlation scores and corrected p-values, formatted for Cytoscape import)
- Executed Jupyter notebook with R kernel session

## How to apply

Install R first, then register the native R kernel for Jupyter by installing the IRkernel package from CRAN and calling IRkernel::installspec() to register the kernel ('ir' / 'R') in the current R installation. Use Anaconda as the environment manager to obtain Jupyter and manage Python/R dependencies together. Launch the notebook downloaded from github.com/DorresteinLaboratory/Bioactive_Molecular_Networks, select the R kernel from the Jupyter dropdown, and run it on the bioassay-annotated feature table (features_quantification_matrix_edited_bioactivity.csv) transposed so features are rows and LC-MS/MS fractions are columns. The notebook computes, per feature, the Pearson correlation between its intensity profile across fractions and the bioassay result vector, applies multiple-testing p-value correction, and writes features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv formatted as a Cytoscape node attribute table. Rationale: the authors released the code as a Jupyter notebook explicitly for easiness and reproducibility, so execution inside the notebook environment preserves the exact published analysis.

## Related tools

- **IRkernel** (Native R kernel that lets Jupyter execute the notebook's R correlation code) — https://github.com/IRkernel/IRkernel
- **Jupyter Notebook** (Web application hosting and running the Step 2 bioactivity-scoring notebook)
- **R** (Language in which the Pearson correlation and p-value computation are implemented)
- **Anaconda** (Convenient environment manager for installing Jupyter and dependency stacks)

## Examples

```
install.packages('IRkernel'); IRkernel::installspec()
```

## Evaluation signals

- An 'ir' (R) kernel appears in the Jupyter launcher dropdown after IRkernel::installspec() registration.
- Notebook cells execute end-to-end without kernel-not-found errors and without Python/R package import failures.
- Output file features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv is written and contains one correlation score and corrected p-value per detected feature.
- Corrected p-values reflect the applied multiple-testing correction and correlation coefficients lie within [-1, 1].
- The output table schema matches what GNPS FBMN/Cytoscape expects for a node attribute table.

## Limitations

- The notebook explicitly requires a prior R installation plus IRkernel registration; without it, the notebook cannot execute.
- The workflow recommends Anaconda as the environment manager, suggesting dependency management outside Anaconda can be error-prone.
- Binder is offered as an alternative to local installation, implying local setup friction (OS/dependency variability) is a known pain point.

## Evidence

- [readme] IRkernel::installspec()  # to register the kernel in the current R installation: [evidence span withheld — anchor status: external_doc]
