---
name: bioactivity-score-node-mapping
description: Use when a feature quantification table with fraction-wise bioassay values
  is available and you need to prioritize ions in a molecular network by correlation
  with measured bioactivity rather than by abundance alone.
when_to_use_negative:
- The feature table has not yet been correlated against bioassay values — running
  the notebook (Step 2, Pearson correlation calculation) is a prerequisite before
  any node mapping.
- The molecular network does not yet exist, e.g. the .MGF has not been submitted to
  a GNPS FBMN Data Analysis job and imported into Cytoscape.
- The workflow relies on aligned LC-MS/MS features across fractions; it is not applicable
  when no fraction-wise feature quantification (e.g. MZmine2 'Peak area' exports)
  is available.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
tools:
- name: MZmine2
  role: Produces the aligned feature quantification table and .MGF MS/MS summary from
    LC-MS/MS fraction data that feed the correlation calculation and the GNPS FBMN
    network
  repo: http://mzmine.github.io/
- name: Jupyter notebook
  role: Executes the Pearson correlation calculation (Step 2) generating the significance
    table used as node attributes
  repo: https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- name: GNPS web-platform
  role: Runs the feature-based molecular networking (FBMN) Data Analysis job that
    generates the molecular network whose nodes receive the bioactivity scores
  repo: http://gnps.ucsd.edu
- name: Cytoscape
  role: Imports the network and joins the correlation table as node attributes; selection
    and styling (r² > 0.8, p-value < -0.02) produce the annotated bioactive molecular
    network
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jnatprod.7b00737
    title: Bioactivity-Based Molecular Networking for the Discovery of Drug Leads
      in Natural Product Bioassay-Guided Fractionation
  source_content_hash: sha256:64dbbb8362161e011d412a7007a844433c80bb0549159c77355753c43901156c
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - Cytoscape
  - GNPS web-platform
  - Jupyter notebook
  - MZmine2
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# bioactivity-score-node-mapping

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Maps Pearson-correlation bioactivity scores from a fraction-bioassay dataset onto molecular network nodes in Cytoscape, so candidate bioactive ions (e.g. antiviral hits in Euphorbia dendroides) become visually distinguishable features in the GNPS feature-based molecular network.

## When to use

Use when a feature quantification table with fraction-wise bioassay values is available and you need to prioritize ions in a molecular network by correlation with measured bioactivity rather than by abundance alone.

## When NOT to use

- The feature table has not yet been correlated against bioassay values — running the notebook (Step 2, Pearson correlation calculation) is a prerequisite before any node mapping.
- The molecular network does not yet exist, e.g. the .MGF has not been submitted to a GNPS FBMN Data Analysis job and imported into Cytoscape.
- The workflow relies on aligned LC-MS/MS features across fractions; it is not applicable when no fraction-wise feature quantification (e.g. MZmine2 'Peak area' exports) is available.

## Inputs

- GNPS FBMN molecular network imported into Cytoscape (.cys) built from the .MGF MS/MS summary file exported by MZmine2
- Jupyter notebook output table features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv (Pearson correlation scores with significance per feature)
- Feature quantification matrix edited with bioassay values (features_quantification_matrix_edited_bioactivity.csv)

## Outputs

- Cytoscape session bioactive_molecular_network.cys in which nodes with r² > 0.8 and p-value < -0.02 are visually distinguishable by size, shape, and/or piechart annotation
- Exported rendered view of the bioactive-annotated molecular network

## How to apply

After the Jupyter notebook computes a Pearson correlation score and its significance for every detected feature against the bioassay results, import the output table features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv into Cytoscape (version ≥ 3.4) as a node attribute table joined to row-ID nodes of the GNPS FBMN network. Select nodes using the threshold r² > 0.8 with p-value < -0.02 (p-value expressed as log10(p)), then restyle those nodes — e.g. resize, reshape, or add piechart diagrams showing each ion's distribution across analyzed samples. The rationale is that score-prediction from relative molecular abundance across fractions combined with per-fraction bioactivity exposes candidates missed by classical bioassay-guided fractionation. Save the annotated session (e.g. bioactive_molecular_network.cys) and export a rendered view; validation relies on visually confirming that significant nodes are distinguishable in the network.

## Related tools

- **MZmine2** (Produces the aligned feature quantification table and .MGF MS/MS summary from LC-MS/MS fraction data that feed the correlation calculation and the GNPS FBMN network) — http://mzmine.github.io/
- **Jupyter notebook** (Executes the Pearson correlation calculation (Step 2) generating the significance table used as node attributes) — https://github.com/DorresteinLaboratory/Bioactive_Molecular_Networks/
- **GNPS web-platform** (Runs the feature-based molecular networking (FBMN) Data Analysis job that generates the molecular network whose nodes receive the bioactivity scores) — http://gnps.ucsd.edu
- **Cytoscape** (Imports the network and joins the correlation table as node attributes; selection and styling (r² > 0.8, p-value < -0.02) produce the annotated bioactive molecular network)

## Examples

```
jupyter notebook Bioactive_Molecular_Networks.ipynb  # run after adding bioassay values to features_quantification_matrix.csv, then import output CSV into Cytoscape
```

## Evaluation signals

- The correlation table features_quantification_matrix_transposed_with_significant_correlation_pvalue_corrected.csv joins to network nodes by row ID so that every feature node carries its Pearson score.
- Nodes flagged as significant use the exact threshold r² > 0.8 combined with p-value < -0.02.
- Selected nodes are restyled (size, shape, or piechart distribution diagrams) so significant vs non-significant nodes are visually distinguishable.
- The styled session is saved as bioactive_molecular_network.cys and a rendered view of the annotated network is exported.
- Node-level significance calls are ultimately subject to expert review per the workflow's validation criteria.

## Limitations

- Exact node-level significance calls are subject to expert review; the visual annotation is a screening aid, not a definitive identification.
- Optimus-based processing requires activation of the MS/MS filter option, and MZmine2 exports require 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area' columns; missing columns break the join.
- The correlation-based bioactivity score presumes per-fraction bioassay values were added to the quantification matrix; without these the mapping is meaningless.
- Putative molecular annotations from mz-RT matching still require MS/MS validation, which Optimus itself does not provide.

## Evidence

- [methods] Node mapping threshold: "The step 2 will calculate the Pearson correlation score and the its significance for every detected features, against the result of the bioassay"
- [intro] Workflow integration rationale: "integrates MS/MS molecular networking and bioassay-guided fractionation into the concept of bioactive molecular networking"
- [methods] Threshold application in Cytoscape: "a table exported with the 'Export to CSV' module in MZmine, that includes 'Export row ID', 'Export row m/z', 'Export row retention time', and 'Peak area'"
- [readme] Repository README workflow: "Add the value of the bioassay experiments for each sample analyzed by LC-MS/MS in the table (features_quantification_matrix.csv to features_quantification_matrix_edited_bioactivity.csv)"
- [abstract] Correlation semantics: "bioactivity score prediction can be calculated with the relative abundance of a molecule in fractions and the bioactivity level of each fraction"
