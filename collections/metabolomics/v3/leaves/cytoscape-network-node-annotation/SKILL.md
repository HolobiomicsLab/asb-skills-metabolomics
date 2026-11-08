---
name: cytoscape-network-node-annotation
description: Use when a GNPS FBMN job has produced a molecular network (imported into
  Cytoscape ≥3.4) and the notebook output table of Pearson correlation scores and
  corrected p-values per feature must be joined to network nodes to flag significant
  bioactivity.
when_to_use_negative:
- The LC-MS/MS data has not yet been processed into a feature quantification table
  and .MGF MS/MS summary (run MZmine2/Optimus and the GNPS FBMN job first)
- No bioassay values are available for the fractions analyzed by LC-MS/MS, so Pearson
  correlation scores against bioactivity cannot be computed
- The molecular network was generated via MS-Cluster instead of the feature-based
  molecular networking (FBMN) workflow, so feature row IDs needed for the table join
  are absent
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: Cytoscape
  role: Imports the GNPS FBMN molecular network, joins the correlation CSV as a node
    attribute table by row ID, selects nodes with r² > 0.8 and p-value < -0.02, styles
    them, and saves the .cys session
- name: GNPS web-platform
  role: Hosts the feature-based molecular networking (FBMN) Data Analysis job that
    generates the network from the MZmine2-exported .MGF file
  repo: http://gnps.ucsd.edu
- name: Jupyter notebook
  role: Produces the per-feature Pearson correlation score and corrected p-value table
    that becomes the Cytoscape node attribute table
  repo: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- name: MZmine2
  role: Exports the aligned feature table and .MGF MS/MS spectral summary consumed
    by the FBMN job whose network is annotated
  repo: http://mzmine.github.io/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:760d53ecda0493bcf97391e20351d433d794aabd8ae5a5be31ebc189d630a3f5
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Cytoscape
  - GNPS web-platform
  - Jupyter notebook
  - MZmine2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cytoscape-network-node-annotation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Imports per-feature bioactivity correlation results (Pearson r² and p-value from the bioactive molecular networking Jupyter notebook) into Cytoscape as node attributes on a GNPS feature-based molecular network, then styles and exports the network so bioactive candidate nodes are visually distinguishable.

## When to use

Use when a GNPS FBMN job has produced a molecular network (imported into Cytoscape ≥3.4) and the notebook output table of Pearson correlation scores and corrected p-values per feature must be joined to network nodes to flag significant bioactivity.

## When NOT to use

- The LC-MS/MS data has not yet been processed into a feature quantification table and .MGF MS/MS summary (run MZmine2/Optimus and the GNPS FBMN job first)
- No bioassay values are available for the fractions analyzed by LC-MS/MS, so Pearson correlation scores against bioactivity cannot be computed
- The molecular network was generated via MS-Cluster instead of the feature-based molecular networking (FBMN) workflow, so feature row IDs needed for the table join are absent

## Inputs

- GNPS FBMN Data Analysis job results (molecular network generated from the MZmine2-exported .MGF MS/MS spectral summary)
- features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv (Jupyter notebook output joining Pearson correlation scores and corrected p-values to feature row IDs)
- Cytoscape session of the imported molecular network (Cytoscape minimum version 3.4)

## Outputs

- bioactive_molecular_network.cys — saved Cytoscape session with bioactivity-annotated, styled nodes
- Rendered exported view of the bioactive molecular network with significant nodes visually distinguishable (size/shape/piechart)

## How to apply

Run the GNPS feature-based molecular networking (FBMN) Data Analysis job from the .MGF file exported by MZmine2 (rather than via MS-Cluster), download the results, and import the network into Cytoscape (minimum version 3.4) following the GNPS documentation. Import the notebook output table features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv as a node attribute table, joining Pearson correlation scores to nodes by feature row ID. Use Cytoscape's Select function to flag nodes meeting the significance threshold r² > 0.8 and p-value < -0.02, then bypass the style of the selected nodes (adjust node size or shape) and optionally add piecharts showing each ion's distribution across the analyzed samples. Save the styled session as bioactive_molecular_network.cys and export a rendered view of the bioactive molecular network. Rationale: mapping the notebook's bioactivity scores onto network nodes exposes potentially bioactive molecules that classical bioassay-guided fractionation missed in the Euphorbia dendroides case.

## Related tools

- **Cytoscape** (Imports the GNPS FBMN molecular network, joins the correlation CSV as a node attribute table by row ID, selects nodes with r² > 0.8 and p-value < -0.02, styles them, and saves the .cys session)
- **GNPS web-platform** (Hosts the feature-based molecular networking (FBMN) Data Analysis job that generates the network from the MZmine2-exported .MGF file) — http://gnps.ucsd.edu
- **Jupyter notebook** (Produces the per-feature Pearson correlation score and corrected p-value table that becomes the Cytoscape node attribute table) — https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- **MZmine2** (Exports the aligned feature table and .MGF MS/MS spectral summary consumed by the FBMN job whose network is annotated) — http://mzmine.github.io/

## Evaluation signals

- The correlation table is joined to network nodes such that feature row IDs match the network node attributes (verified by inspecting node tables in the Cytoscape session)
- Nodes meeting the threshold r² > 0.8 and p-value < -0.02 are visually distinguishable from the rest of the network by node size, shape, or piechart
- A saved Cytoscape session file named bioactive_molecular_network.cys exists and reopens with the bioactivity annotations intact
- An exported rendered view of the bioactive molecular network shows the styled, bioactivity-annotated nodes
- Exact node-level significance calls are subject to expert review before the annotated network is accepted

## Limitations

- Node-level significance calls (r² > 0.8, p-value < -0.02) are provisional and subject to expert review; the annotation is validated by visual inspection of the saved session, not automatically
- The join depends on the FBMN workflow: the .MGF must be submitted to GNPS feature-based molecular networking rather than MS-Cluster so feature row IDs exist for the node attribute table
- The MZmine2 export must use the required Peak list row filter options ('Keep only features with MS/MS scan (GNPS)' and 'Reset the peak number') or features without MS/MS scans will lack network nodes to annotate

## Evidence
