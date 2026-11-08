---
name: mass-action-law-based-substrate-availability-quantification
description: Decision support for converting intracellular metabolite abundance tables
  (e.g. metabolomics_LM.csv) into reaction-level substrate-availability scores when
  mapping substrate abundance variation to metabolic flux differences via the mass
  action law.
when_to_use_negative:
- When the goal is to quantify flux changes due to transcriptional regulation of enzyme
  genes (that is the RAS/GPR pathway), not substrate-availability control.
- When extracellular metabolomics data are the only available abundance information
  — the mass action law in INTEGRATE is applied to intracellular substrate concentrations.
- When one or more substrates of the reaction of interest are missing from the metabolomics
  quantification; such reactions are dropped rather than imputed.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: INTEGRATE
  role: Pipeline that implements the mass action law formulation on intracellular
    metabolomics to predict how differences in substrate availability translate into
    flux differences.
  repo: https://github.com/qLSLab/integrate
- name: createMetabolicDataset.py
  role: Pipeline step that builds the metabolomic statistical dataset consumed for
    RPS-based concordance analysis.
  repo: https://github.com/qLSLab/integrate
- name: concordanceAnalysis.py
  role: Downstream step computing Cohen kappa and Pearson correlation of RPS vs RAS
    and RPS vs FFD to classify metabolic vs transcriptional regulation.
  repo: https://github.com/qLSLab/integrate
- name: Python
  role: Runtime for executing the INTEGRATE pipeline scripts that compute and normalize
    RPS.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:66e6d8ea39e4f03debebc38ce61aa1ff18b31d45ac03495e8f0e61675cbcb13e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qLSLab/integrate
  tools:
  - INTEGRATE
  - Python
  - concordanceAnalysis.py
  - createMetabolicDataset.py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mass-action-law-based-substrate-availability-quantification

## Summary

Computation of a Reaction Propensity Score (RPS) from intracellular metabolomics data by raising each substrate concentration to the power of its stoichiometric coefficient in the (irreversible) metabolic model and multiplying them, following the mass action law, to quantify substrate-availability differences between cell lines.

## When to use

Decision support for converting intracellular metabolite abundance tables (e.g. metabolomics_LM.csv) into reaction-level substrate-availability scores when mapping substrate abundance variation to metabolic flux differences via the mass action law.

## When NOT to use

- When the goal is to quantify flux changes due to transcriptional regulation of enzyme genes (that is the RAS/GPR pathway), not substrate-availability control.
- When extracellular metabolomics data are the only available abundance information — the mass action law in INTEGRATE is applied to intracellular substrate concentrations.
- When one or more substrates of the reaction of interest are missing from the metabolomics quantification; such reactions are dropped rather than imputed.

## Inputs

- Intracellular metabolomics dataset of metabolite abundances per cell line (metabolomics_LM.csv)
- Irreversible stoichiometric metabolic model in SBML format (ENGRO2_irrev.xml) providing reaction-substrate stoichiometry
- Conversion dictionary mapping metabolomics metabolite IDs to model metabolite IDs (metsEngroVsMetabolomics.csv)
- List of sample/cell line names (e.g. ['MCF102A','SKBR3','MCF7','MDAMB231','MDAMB361'])

## Outputs

- Reaction Propensity Score (RPS) dataset: one RPS value per eligible reaction per sample, restricted to reactions whose substrates are all quantified (81 reactions for ENGRO2/breast cell lines)
- Per-cell-line normalized RPS values used downstream for concordance analysis with RAS and FFD datasets
- Intermediate file of computed substrate means per cell line (medie_Met.csv)

## How to apply

Load the irreversible metabolic model (e.g. ENGRO2_irrev.xml) and extract for every reaction the stoichiometric coefficient of each substrate. Convert metabolite identifiers from the metabolomics dataset to model identifiers using a conversion dictionary (e.g. metsEngroVsMetabolomics.csv). Retain only reactions for which all substrates are quantified in the intracellular metabolomics data, discarding any reaction with at least one missing substrate. For each retained reaction and each cell line (MCF102A, SKBR3, MCF7, MDAMB231, MDAMB361), compute the RPS as the product of substrate concentrations each raised to the power of its stoichiometric coefficient, neglecting enzymatic activity. Normalize per-cell-line RPS values across the eligible reactions (81 for the ENGRO2 breast-cell-line application) so that relative differences in substrate availability become comparable between samples and can be used downstream for concordance analysis (RPS vs RAS, RPS vs FFD).

## Related tools

- **INTEGRATE** (Pipeline that implements the mass action law formulation on intracellular metabolomics to predict how differences in substrate availability translate into flux differences.) — https://github.com/qLSLab/integrate
- **createMetabolicDataset.py** (Pipeline step that builds the metabolomic statistical dataset consumed for RPS-based concordance analysis.) — https://github.com/qLSLab/integrate
- **concordanceAnalysis.py** (Downstream step computing Cohen kappa and Pearson correlation of RPS vs RAS and RPS vs FFD to classify metabolic vs transcriptional regulation.) — https://github.com/qLSLab/integrate
- **Python** (Runtime for executing the INTEGRATE pipeline scripts that compute and normalize RPS.)

## Examples

```
python pipeline/createMetabolicDataset.py  # builds metabolomic dataset for concordance analysis using ENGRO2_irrev.xml and metabolomics_LM.csv
```

## Evaluation signals

- Reaction count invariant: the output contains RPS values for exactly 81 reactions (those with all substrates quantified) and none for reactions with any missing substrate.
- Identifier consistency: every metabolite used in the product has been mapped through metsEngroVsMetabolomics.csv to a valid model metabolite ID in ENGRO2_irrev.xml.
- Exponent correctness: each substrate concentration is raised to its stoichiometric coefficient taken from the irreversible model, not to an arbitrary power or the same exponent for all substrates.
- Per-sample completeness: RPS values exist for every input cell line (MCF102A, SKBR3, MCF7, MDAMB231, MDAMB361) with no missing cells in the eligible reaction rows.
- Downstream concordance produces finite Cohen kappa (linear weighted) and Pearson correlation values when RPS is compared with RAS and FFD, without undefined ratios (eps=0.00001 used in the pipeline).

## Limitations

- Reactions with at least one missing substrate quantification are omitted; the resulting RPS dataset is therefore incomplete (only 81 reactions for ENGRO2 breast-cell-line data) and coverage depends entirely on metabolomics detection.
- Enzymatic activity is neglected in the mass action law formulation; RPS does not capture allosteric, post-translational or enzyme-level regulation of flux.
- The score is a relative, model-scaffolded proxy for substrate-availability flux differences, not an absolute thermodynamic propensity; it requires an irreversible model whose stoichiometry matches the mapped metabolites.

## Evidence

- [intro] Mass action law formulation: "INTEGRATE uses intracellular metabolomics datasets and the mass action law formulation to predict how differences in substrate availability translate into differences in metabolic fluxes"
- [methods] Implementation step: [evidence span withheld — anchor status: not_found]
- [readme] Model and data inputs: "metabolic_model: the input model name. Default value: 'ENGRO2_irrev.xml'  * metabolic_data: dataset of metabolites. Default value: 'metabolomics_LM.csv'"
- [abstract] Discrimination rationale: "We discriminate fluxes regulated at the metabolic and/or gene expression level by intersecting these two output datasets"
