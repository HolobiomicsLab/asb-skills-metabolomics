---
name: cytoscape-bioactivity-node-visualization
description: Use when you have run the Bioactive_Molecular_Networks Jupyter notebook
  and submitted the .MGF plus feature table to GNPS FBMN, and you need to expose,
  style, and inspect the statistically correlated bioactive candidate nodes inside
  Cytoscape.
when_to_use_negative:
- You only have the raw LC-MS/MS data and no feature quantification table with bioassay
  values — the network cannot yet be imported or styled.
- The analysis goal is classical bioassay-guided fractionation without molecular networking,
  since this skill assumes the FBMN job and correlation table already exist.
- You want to visualize a standard (non-bioactive) FBMN network without overlaying
  correlation scores — the significance threshold and bypass styling steps are specific
  to the bioactivity mapping.
edam_operation: http://edamontology.org/operation_3434
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Cytoscape
  role: Import the GNPS molecular network and the correlation table, flag bioactive
    nodes via Select with r2 > 0.8 / p < 0.02, apply bypass node style and pie charts
    of ion distribution.
- name: GNPS web-platform
  role: Generates the feature-based molecular networking job whose output network
    is imported into Cytoscape.
  repo: http://gnps.ucsd.edu
- name: Bioactive_Molecular_Networks Jupyter notebook
  role: Produces the correlation table (r2 and corrected p-values) that is imported
    into Cytoscape for node flagging.
  repo: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- name: MZmine2
  role: Exports the feature table (.csv with row ID, row m/z, row retention time,
    peak area) and .MGF spectral summary that feed the GNPS job underlying the network.
  repo: http://mzmine.github.io/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:1ccf7ed2dba1dfa357fd7bdd44d19977268d1cf42cad2e5e14ffdd7b5c57cadc
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Bioactive_Molecular_Networks Jupyter notebook
  - Cytoscape
  - GNPS web-platform
  - MZmine2
  - OpenMS
  - Optimus
  - R
  edam_operation: http://edamontology.org/operation_3434
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cytoscape-bioactivity-node-visualization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Visualize the bioactive molecular network in Cytoscape by importing the GNPS FBMN network plus the notebook's Pearson correlation table, flagging features with r2 > 0.8 and p < 0.02 as bioactive nodes and styling them by node size/shape with pie charts of sample distribution.

## When to use

Use when you have run the Bioactive_Molecular_Networks Jupyter notebook and submitted the .MGF plus feature table to GNPS FBMN, and you need to expose, style, and inspect the statistically correlated bioactive candidate nodes inside Cytoscape.

## When NOT to use

- You only have the raw LC-MS/MS data and no feature quantification table with bioassay values — the network cannot yet be imported or styled.
- The analysis goal is classical bioassay-guided fractionation without molecular networking, since this skill assumes the FBMN job and correlation table already exist.
- You want to visualize a standard (non-bioactive) FBMN network without overlaying correlation scores — the significance threshold and bypass styling steps are specific to the bioactivity mapping.

## Inputs

- features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv (notebook output table with Pearson r2 and corrected p-values)
- GNPS FBMN molecular network output (network file from a Data Analysis job on gnps.ucsd.edu)
- Analyzed fractions with measured bioassay values (e.g. antiviral activity for Euphorbia dendroides LC-MS/MS fractions)
- .MGF MS/MS spectral summary and features_quantification_matrix.csv already submitted to GNPS

## Outputs

- Cytoscape molecular network where nodes with r2 > 0.8 and p < 0.02 are flagged and visually styled as bioactive candidates
- Pie-chart node annotations showing ion distribution of each feature across samples
- Dereplicated candidate list of bioactive leads (e.g. antiviral molecules missed by classical bioassay-guided fractionation)

## How to apply

Import the GNPS feature-based molecular networking job output into Cytoscape (minimum version 3.4), then import the correlation output table features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv into the same session. Use the Cytoscape Select function with the significance threshold r2 > 0.8 and p-value < 0.02 to flag bioactive candidate nodes among the molecular network. Apply the bypass node style (size and shape) so selected bioactive nodes are visually distinguished from the rest of the network, and display pie charts on nodes showing the ion distribution of each feature across the analyzed samples. Rationale: the Pearson correlation score computed in the Jupyter notebook predicts bioactivity from molecule relative abundance in fractions against the bioassay values, and the network visualization is the point where statistically supported candidates become dereplicable and actionable for isolation.

## Related tools

- **Cytoscape** (Import the GNPS molecular network and the correlation table, flag bioactive nodes via Select with r2 > 0.8 / p < 0.02, apply bypass node style and pie charts of ion distribution.)
- **GNPS web-platform** (Generates the feature-based molecular networking job whose output network is imported into Cytoscape.) — http://gnps.ucsd.edu
- **Bioactive_Molecular_Networks Jupyter notebook** (Produces the correlation table (r2 and corrected p-values) that is imported into Cytoscape for node flagging.) — https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- **MZmine2** (Exports the feature table (.csv with row ID, row m/z, row retention time, peak area) and .MGF spectral summary that feed the GNPS job underlying the network.) — http://mzmine.github.io/

## Examples

```
Jupyter notebook (R kernel): Bioactive_molecular_networks_v1.0.ipynb → GNPS FBMN job → Cytoscape 3.4: Select nodes where r2 > 0.8 & p < 0.02 → apply bypass node style → display pie charts of ion distribution across samples
```

## Evaluation signals

- The correlation output table contains a corrected p-value column and the Pearson r2 column for every detected feature.
- Cytoscape selection with r2 > 0.8 and p < 0.02 recovers the antiviral candidate nodes reported for the Euphorbia dendroides extract (candidates missed by classical bioassay-guided fractionation).
- Bypass node style makes the flagged bioactive nodes distinguishable in size/shape from the non-bioactive network.
- Node pie charts match the relative abundance of each feature across the analyzed fractions as recorded in features_quantification_matrix_edited_bioactivity.csv.
- The Cytoscape network layout is consistent with the GNPS FBMN job output (matching edge/node count and ion identity edges).

## Limitations

- Requires Cytoscape minimum version 3.4 for the FBMN import and styling.
- The Pearson correlation score and significance are computed for every detected feature against the bioassay result; features with insufficient bioassay variation cannot be meaningfully correlated.
- The network is a dereplication aid — MS/MS molecular networking exposes candidates prior to subsequent isolation of the compounds; follow-up isolation is still needed.
- The notebook and workflow were validated on a previously investigated Euphorbia dendroides extract; other bioassay types or fraction designs may require adapting the significance thresholds.

## Evidence

- [methods] Cytoscape minimum version 3.4: [evidence span withheld — anchor status: not_found]
