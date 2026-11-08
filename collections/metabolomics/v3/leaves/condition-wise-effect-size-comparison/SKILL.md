---
name: condition-wise-effect-size-comparison
description: Use when a lipid abundance matrix spans multiple stimulation conditions
  (e.g. 0.01/1 U/mL thrombin, 1/5 µg/mL CRP) plus an unstimulated control, and the
  goal is to rank which lipid classes respond to stimulation and which stimulus is
  the stronger regulator.
when_to_use_negative:
- Input is already an aggregated per-class fold-change table rather than a species-level
  abundance matrix, since per-species log2FC and headgroup grouping must be re-derived
  from raw abundances.
- The aim is de novo lipid identification or spectral library construction rather
  than differential comparison — use LipidCreator's assay-design features instead
  of effect-size comparison.
- Unstimulated control data are absent, making relative fold-change computation against
  a baseline impossible.
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: LipidCreator
  role: Designs the targeted PRM/MRM transition lists and in silico spectral libraries
    that yield the platelet lipid abundance matrix used in the fold-change analysis.
  repo: https://github.com/lifs-tools/lipidcreator
- name: Skyline
  role: Hosts LipidCreator as a plugin for targeted lipidomics method preparation
    and quantitative peak-area data analysis.
  repo: https://skyline.ms/project/home/software/Skyline/begin.view
- name: MSStats
  role: Statistical analysis of the targeted quantitative data underlying condition-wise
    comparisons.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:610e1c6177d02b697eb397c036fda8cebd3c3cbade89d63d1f8ddb4d5603baa8
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Condition-wise effect-size comparison of lipid classes across stimulation states

## Summary

Quantifies stimulation-driven lipid remodeling by computing log2 fold-changes per lipid species against an unstimulated control, aggregating them to mean absolute effect sizes per lipid class (PI, PC, PE, PS, PG), and testing whether one class (e.g. PI species like PI 38:4) dominates the arachidonic-acid pool in platelet lipid signaling.

## When to use

Use when a lipid abundance matrix spans multiple stimulation conditions (e.g. 0.01/1 U/mL thrombin, 1/5 µg/mL CRP) plus an unstimulated control, and the goal is to rank which lipid classes respond to stimulation and which stimulus is the stronger regulator.

## When NOT to use

- Input is already an aggregated per-class fold-change table rather than a species-level abundance matrix, since per-species log2FC and headgroup grouping must be re-derived from raw abundances.
- The aim is de novo lipid identification or spectral library construction rather than differential comparison — use LipidCreator's assay-design features instead of effect-size comparison.
- Unstimulated control data are absent, making relative fold-change computation against a baseline impossible.

## Inputs

- Lipid abundance matrix (lipid species as rows; 0.01 U/mL thrombin, 1 U/mL thrombin, 1 µg/mL CRP, 5 µg/mL CRP, and unstimulated control as columns)
- Lipid class headgroup annotations (PI, PC, PE, PS, PG) and AA-content annotations per species
- Targeted lipidomics quantification from a LipidCreator-designed Skyline assay (PRM/MRM transition data)

## Outputs

- Per-species log2 fold-change table relative to the unstimulated control
- Mean absolute log2 fold-change per lipid class per stimulation condition
- Spearman correlation network (edges with |rho| > 0.7, FDR-adjusted p < 0.05)
- AA-reservoir ranking showing whether PI (e.g. PI 38:4) carries the largest AA-containing fraction
- CRP vs thrombin regulatory-strength ranking by condition-wise effect sizes

## How to apply

Load the lipid abundance matrix (rows = lipid species incl. PI, PC, PE, PS, PG and AA-containing species; columns = the four stimulation conditions plus control) with pandas, compute per-species log2 fold-changes relative to the unstimulated control for symmetric comparison, then group species by headgroup class and compute the mean absolute log2 fold-change per class per condition. Build a Spearman rank correlation network across all species pairs, flagging edges with |rho| > 0.7 and FDR-adjusted p < 0.05 to confirm that PI species cluster together. Within the PI class, compare the relative abundance of AA-containing species (PI 38:4 and related) against other PI species to verify PI as the dominant AA reservoir, and compute condition-wise effect sizes to rank CRP versus thrombin regulatory strength at matched stimulus intensities. Quantification of the underlying targeted assay data is prepared with LipidCreator/Skyline and analyzed with MSStats.

## Related tools

- **LipidCreator** (Designs the targeted PRM/MRM transition lists and in silico spectral libraries that yield the platelet lipid abundance matrix used in the fold-change analysis.) — https://github.com/lifs-tools/lipidcreator
- **Skyline** (Hosts LipidCreator as a plugin for targeted lipidomics method preparation and quantitative peak-area data analysis.) — https://skyline.ms/project/home/software/Skyline/begin.view
- **MSStats** (Statistical analysis of the targeted quantitative data underlying condition-wise comparisons.)

## Examples

```
import pandas as pd; df = pd.read_csv('platelet_lipidome.csv', index_col=0); log2fc = (df[['thrombin_0.01U','thrombin_1U','crp_1ug','crp_5ug']].sub(df['control'], axis=0)).div(df['control'], axis=0).apply(np.log2)
```

## Evaluation signals

- Re-derived mean |log2FC| for PI species is substantially larger than that for PC/PE, directionally consistent with 'dramatically reduced' vs 'unaffected'.
- PI carries the largest AA-containing fraction (e.g. PI 38:4 and related species dominate the AA pool).
- CRP-induced mean |log2FC| exceeds thrombin-induced mean |log2FC| at matched stimulus intensities.
- The reconstructed correlation network reproduces PI-species clustering with edges at |rho| > 0.7 and FDR-adjusted p < 0.05.
- Direct correlation analysis of quantitative reproducibility reaches agreement on the order of R2 = 0.98, as reported for reference vs control samples in the article.

## Limitations

- LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates and intentionally does not address coeluting/isobaric lipids and peak area integration, so FDR must be handled in downstream statistics.
- Retention time, elution order, and isotope labeled standards need to be considered to avoid mis-annotations of lipid species.
- Collision energy models are currently exclusive to lipid mediators; other lipid classes require structurally diverse standards and CE-dependent measurements.
- The current implementation focuses on eukaryotic model systems due to lack of standards for bacteria such as E. coli and archaea.

## Evidence

- [abstract] LipidCreator assays were validated computationally and analytically and applied to blood analysis and platelet lipid-signaling pathways: "We validate LipidCreator assays computationally and analytically and prove that it is capable to generate large targeted experiments to analyze blood and to dissect lipid-signaling pathways such as"
- [discussion] Direct correlation analysis reveals R2 = 0.98 between reference material and control group: "a direct correlation analysis reveals a good agreement (R2 = 0.98) between the American reference material and our Asian control group"
- [discussion] No dedicated FDR scoring; coeluting/isobaric lipids and peak integration not addressed: "LipidCreator does not provide a dedicated scoring algorithm to determine false discovery rates but supports a knowledge-based selection of transition lists."
- [readme] LipidCreator is a Skyline plugin producing target lists and fragment libraries for PRM and MRM experiments: "It can be used to create user-defined target lists and fragment libraries for PRM and MRM experiments in Skyline."
