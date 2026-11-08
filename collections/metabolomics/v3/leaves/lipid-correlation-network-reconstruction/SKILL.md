---
name: lipid-correlation-network-reconstruction
description: Use when you have a targeted-lipidomics abundance matrix (lipid species
  such as PI, PC, PE, PS, PG and AA-containing species as rows; e.g. 0.01/1 U/mL thrombin,
  1/5 µg/mL CRP plus unstimulated control as columns) and need to test class-level
  depletion claims and co-regulation structure.
when_to_use_negative:
- The input is already a correlation network or edge list — no abundance matrix to
  re-correlate.
- The dataset has no unstimulated control column, so per-condition log2 fold-changes
  cannot be computed.
- Lipid identifications come from unvalidated assays in simple model systems (e.g.
  yeast) without multiple fragments and both polarity modes, making co-regulation
  signals unreliable.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: LipidCreator
  role: Designs the targeted lipidomics assays and fragment/transition lists for the
    PI/PC/PE/PS/PG and AA-containing species quantified before network reconstruction;
    fully integrated with Skyline
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Hosts the LipidCreator assays and performs method preparation and data analysis
    for the targeted PRM/SRM lipid quantification feeding the correlation network
- name: MSStats
  role: Statistical analysis layer for the quantitative lipidomics results used in
    effect-size comparisons across stimulation conditions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:c3563be9f926e5bd99fd74b7b21e02de1c01cb893f6f21fe099222a8c87c15d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - MSStats
  - Skyline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# lipid-correlation-network-reconstruction

## Summary

Reconstructs a Spearman rank-correlation network over platelet lipid species and class-level log2 fold-change summaries across thrombin/CRP stimulation conditions, to test claims about PI-cluster depletion and the PI pool as the dominant arachidonic-acid (AA) reservoir. Uses Skyline/LipidCreator-derived targeted-lipidomics quantities as input.

## When to use

Use when you have a targeted-lipidomics abundance matrix (lipid species such as PI, PC, PE, PS, PG and AA-containing species as rows; e.g. 0.01/1 U/mL thrombin, 1/5 µg/mL CRP plus unstimulated control as columns) and need to test class-level depletion claims and co-regulation structure.

## When NOT to use

- The input is already a correlation network or edge list — no abundance matrix to re-correlate.
- The dataset has no unstimulated control column, so per-condition log2 fold-changes cannot be computed.
- Lipid identifications come from unvalidated assays in simple model systems (e.g. yeast) without multiple fragments and both polarity modes, making co-regulation signals unreliable.

## Inputs

- Lipid abundance matrix (lipid species as rows; stimulation conditions 0.01 U/mL thrombin, 1 U/mL thrombin, 1 ug/mL CRP, 5 ug/mL CRP plus unstimulated control as columns)
- Skyline-exported transition-level peak areas from PRM/SRM targeted assays (Q-Exactive HF or QTRAP 6500)
- Lipid-to-headgroup class mapping (PI, PC, PE, PS, PG) with AA-containing species annotated (e.g. PI 38:4)

## Outputs

- Per-species log2 fold-change table relative to the unstimulated control
- Class-level mean absolute log2 fold-change summary per lipid class per condition
- Spearman correlation network edge list with |rho| > 0.7 and FDR-adjusted p < 0.05 flags
- AA-reservoir ranking (AA-containing PI fraction vs other PI species) and CRP-vs-thrombin effect-size ranking

## How to apply

Load the lipid abundance matrix with pandas, compute per-species log2 fold-changes for each stimulation condition relative to the unstimulated control, then group species by lipid class headgroup (PI, PC, PE, PS, PG) and compute mean absolute log2 fold-change per class per condition. Compute Spearman rank correlation coefficients and p-values across all lipid species pairs and flag network edges with |rho| > 0.7 and FDR-adjusted p < 0.05. Within the PI class, compare relative abundance of AA-containing species (PI 38:4 and related) against other PI species to verify PI as the dominant AA reservoir, and compute condition-wise effect sizes to rank CRP vs thrombin regulatory strength. Quantitative peak areas should come from validated Skyline/LipidCreator targeted assays (PRM or SRM on Q-Exactive HF or QTRAP 6500), with MSStats used for the downstream statistical layer. Validation hinges on directional consistency: PI species should show substantially larger mean |log2FC| than PC/PE, and the re-derived network should reproduce PI-species clustering.

## Related tools

- **LipidCreator** (Designs the targeted lipidomics assays and fragment/transition lists for the PI/PC/PE/PS/PG and AA-containing species quantified before network reconstruction; fully integrated with Skyline) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Hosts the LipidCreator assays and performs method preparation and data analysis for the targeted PRM/SRM lipid quantification feeding the correlation network)
- **MSStats** (Statistical analysis layer for the quantitative lipidomics results used in effect-size comparisons across stimulation conditions)

## Examples

```
import pandas as pd; rho = pd.read_csv('lipid_abundance.csv').set_index('lipid').T.corr(method='spearman')
```

## Evaluation signals

- Re-derived mean |log2FC| for PI species is substantially larger than for PC/PE (directionally consistent with 'dramatically reduced' vs 'unaffected').
- PI carries the largest AA-containing fraction among lipid classes (PI 38:4 and related species dominate the AA pool).
- CRP-induced mean |log2FC| exceeds thrombin at matched stimulus intensities.
- The reconstructed correlation network reproduces PI-species clustering under the |rho| > 0.7, FDR p < 0.05 edge criteria.
- Quantification sanity check: direct correlation analysis between reference and control samples should show high agreement (the paper reports R2 = 0.98 for NIST SRM 1950 vs an Asian control group).

## Limitations

- LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates and intentionally does not address coeluting/isobaric lipids and peak area integration.
- Retention time, elution order of lipids, and isotope labeled standards must be considered to avoid mis-annotations before trusting abundance values.
- Due to lack of isotope-coded standards there are currently no large-scale methods to cope with stereoisomers; LipidCreator CE models are currently exclusive to lipid mediators, and coverage gaps remain for bacterial/archaeal lipids (e.g. acyl-PG, acyl-PE in E. coli).

## Evidence

- [abstract] LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes.: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [intro] software developments such as Skyline15,16, which was further advanced by tools such as Panorama, and MSStats17,18: "software developments such as Skyline15,16, which was further advanced by tools such as Panorama, and MSStats17,18"
- [discussion] a direct correlation analysis reveals a good agreement (R2 = 0.98) between the American reference material and our Asian control group: "a direct correlation analysis reveals a good agreement (R2 = 0.98) between the American reference material and our Asian control group"
- [discussion] DIA method preparation and data analysis were performed with Skyline (See Supplementary Methods).: "DIA method preparation and data analysis were performed with Skyline (See Supplementary Methods)."
- [discussion] additional information such as retention time, the elution order of lipids, as well as isotope labeled standards, if available, always need to be considered in order to avoid mis-annotations.: "additional information such as retention time, the elution order of lipids, as well as isotope labeled standards, if available, always need to be considered in order to avoid mis-annotations."
- [readme] LipidCreator is a plugin for Skyline supporting targeted workflow development in lipidomics.: "LipidCreator is a plugin for [Skyline](https://skyline.ms/project/home/software/Skyline/begin.view) supporting targeted workflow development in lipidomics."
