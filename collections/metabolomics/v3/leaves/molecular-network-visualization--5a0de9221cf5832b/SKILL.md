---
name: molecular-network-visualization
description: Use when an LC-MS/MS feature-based molecular network from GNPS needs
  to be visualized with bioactivity (or other node-level attribute) overlays to prioritize
  candidate molecules.
when_to_use_negative:
- The input is not a GNPS FBMN network already generated from an .MGF file — the skill
  assumes the network has been created via the GNPS web-platform and merely needs
  attribute import and styling.
- No per-node numeric attribute table (e.g. correlation scores keyed on row ID) is
  available to join, since the whole point is overlaying values onto nodes.
- You need dereplication or spectral library matching rather than bioactivity-annotated
  network visualization.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: MZmine2
  role: Processes LC-MS/MS raw data and exports the .MGF file plus CSV feature table
    that feed the GNPS FBMN job used for network generation
  repo: http://mzmine.github.io/
- name: GNPS web-platform
  role: Generates the feature-based molecular network from the submitted .MGF file,
    producing the network results imported into Cytoscape
  repo: http://gnps.ucsd.edu
- name: Cytoscape
  role: Imports the GNPS molecular network, joins correlation-score node attributes,
    selects and styles bioactive nodes (size/shape/piechart), and saves the final
    .cys session
- name: Jupyter notebook
  role: Produces the correlation-score table that is imported into Cytoscape as node
    attributes
  repo: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:d43f4f7a0ed04e8248326326815fb5bb29f1b703ec79fec5e26dbce6a5edb900
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Cytoscape
  - GNPS web-platform
  - Jupyter notebook
  - MZmine2
  - R
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# molecular-network-visualization

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Renders bioactive molecular networks by importing a GNPS feature-based molecular networking job into Cytoscape and overlaying per-feature Pearson correlation scores and significance as node attributes, so that nodes meeting bioactivity thresholds (r² > 0.8, corrected p-value < 0.05) are visually distinguishable.

## When to use

Use when an LC-MS/MS feature-based molecular network from GNPS needs to be visualized with bioactivity (or other node-level attribute) overlays to prioritize candidate molecules.

## When NOT to use

- The input is not a GNPS FBMN network already generated from an .MGF file — the skill assumes the network has been created via the GNPS web-platform and merely needs attribute import and styling.
- No per-node numeric attribute table (e.g. correlation scores keyed on row ID) is available to join, since the whole point is overlaying values onto nodes.
- You need dereplication or spectral library matching rather than bioactivity-annotated network visualization.

## Inputs

- MGF file (.MGF) containing MS/MS spectral summaries exported from MZmine2 or Optimus
- GNPS feature-based molecular networking (FBMN) Data Analysis job results
- Jupyter notebook output table features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv (Pearson correlation scores and corrected p-values per feature row ID)
- Feature quantification table annotated with bioassay values (features_quantification_matrix_edited_bioactivity.csv)

## Outputs

- Bioactive molecular network Cytoscape session saved as bioactive_molecular_network.cys
- Rendered/exported view of the bioactive molecular network with bioactive nodes visually distinguished (size, shape, and/or piechart diagrams showing ion distribution across samples)
- Network in which nodes meeting the r² > 0.8 and p-value < -0.02 threshold are flagged and styled

## How to apply

Submit the .MGF file exported from MZmine2 (version 2.31, using 'Peak list row filter' options 'Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number', plus 'Export to CSV' with 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area') to a GNPS Data Analysis job using the feature-based molecular networking (FBMN) workflow. Download the FBMN results and import the molecular network into Cytoscape (minimum version 3.4) per the GNPS documentation. Join the notebook output table features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv into Cytoscape as a node attribute table keyed on row ID so Pearson correlation scores map to feature nodes. Use the Cytoscape Select function to flag nodes with significant bioactivity using the threshold r² > 0.8 and p-value < -0.02, then bypass the style of the selected nodes (adjust node size or shape) and optionally add piechart diagrams displaying the distribution of each ion across the analyzed samples. Save the styled session as bioactive_molecular_network.cys and export a rendered view of the bioactive molecular network.

## Related tools

- **MZmine2** (Processes LC-MS/MS raw data and exports the .MGF file plus CSV feature table that feed the GNPS FBMN job used for network generation) — http://mzmine.github.io/
- **GNPS web-platform** (Generates the feature-based molecular network from the submitted .MGF file, producing the network results imported into Cytoscape) — http://gnps.ucsd.edu
- **Cytoscape** (Imports the GNPS molecular network, joins correlation-score node attributes, selects and styles bioactive nodes (size/shape/piechart), and saves the final .cys session)
- **Jupyter notebook** (Produces the correlation-score table that is imported into Cytoscape as node attributes) — https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/

## Examples

```
Submit .MGF to GNPS FBMN; download results; Cytoscape: import bioactive_molecular_network.cys; join features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv as node attribute table; Select nodes with r² > 0.8 & p < -0.02; bypass style.
```

## Evaluation signals

- The correlation table (features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv) is joined to network nodes such that nodes meeting the r² > 0.8 and p-value < -0.02 threshold are visually distinguishable (by size/shape/piechart), verified by inspecting the saved Cytoscape session.
- The saved Cytoscape session is named bioactive_molecular_network.cys and contains node attribute values for Pearson correlation scores mapped onto row ID nodes.
- The molecular network topology originates from a GNPS FBMN Data Analysis job submitted with the .MGF file from MZmine2, not from MS-Cluster, confirming the correct upstream workflow was used.
- Node-level significance calls (r² > 0.8, corrected p-value) are subject to expert review after the visual flagging is applied.
- Optional piechart diagrams correctly display the distribution of each ion across the analyzed samples when added.

## Limitations

- The threshold value used for visual flagging is quoted inconsistently in the source cards (p-value < -0.02 vs p-value < 0.05); the notebook applies Pearson correlation with p-value correction, so exact significance calls remain subject to expert review.
- Requires Cytoscape minimum version 3.4 and the GNPS FBMN workflow; it is not applicable to networks generated by other tools or from non-MGF inputs.
- The workflow is specific to LC-MS/MS feature-based molecular networking as described; direct-infusion data or non-OpenMS/MZmine2 preprocessing paths are not supported by this description.

## Evidence

- [methods] Import the notebook output table features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv into Cytoscape as a node attribute table joining Pearson correlation scores to feature (row ID) nodes.: [evidence span withheld — anchor status: not_found]
- [methods] Use the Cytoscape Select function to flag nodes with significant bioactivity using the threshold r2 > 0.8 and p-value < -0.02.: [evidence span withheld — anchor status: not_found]
- [methods] Bypass the style of the selected nodes (adjust node size or shape) and optionally add piechart diagrams displaying the distribution of each ion across the analyzed samples.: [evidence span withheld — anchor status: not_found]
