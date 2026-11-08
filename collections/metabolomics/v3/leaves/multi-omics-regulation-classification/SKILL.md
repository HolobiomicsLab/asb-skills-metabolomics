---
name: multi-omics-regulation-classification
description: Use when you have paired transcriptomics and intracellular metabolomics
  data mapped onto a stoichiometric model and want to determine, per reaction, whether
  flux variation follows substrate abundance (metabolic), enzyme expression (transcriptional),
  both, or neither.
when_to_use_negative:
- The omics data are already integrated and only a single regulation layer is of interest
  — the intersection step adds no value.
- No constraint-based model covering the measured reactions is available; concordance
  analysis requires reaction-level GPRs and flux sampling on the model.
- Only extracellular metabolomics is measured; the paper's mass-action RPS derivation
  requires intracellular substrate abundances.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0153
tools:
- name: INTEGRATE
  role: Full pipeline implementing GPR extraction, RAS computation and normalization,
    RAS/medium/YSI integration into per-cell-line models, flux sampling, Mann-Whitney
    U FFD tests, RAS t-tests, metabolomic statistical-test construction, and concordance
    analysis with linear-weighted Cohen's kappa.
  repo: github.com/QLSLab/integrate
- name: concordanceAnalysis
  role: Computes per-reaction RPS-vs-RAS and RPS-vs-FFD Cohen's kappa (linear weighting)
    and Pearson correlation, p-values and adjusted p-values, and renders the heatmap
    and scatterplot used for regulation classification.
  repo: github.com/QLSLab/integrate
- name: mannWhitneyUTest
  role: Computes Mann-Whitney U tests on randomly sampled steady-state flux solutions
    per cell-line pair to obtain FFD significance feeding the concordance analysis.
  repo: github.com/QLSLab/integrate
- name: createMetabolicDataset
  role: Builds the metabolomic statistical-test dataset (quality filter 1, valLog=1.2)
    and per-pair log2 mean ratios used in the concordance analysis.
  repo: github.com/QLSLab/integrate
- name: rasTtest
  role: Runs pairwise t-tests on RAS scores for each of the 10 cell-line pairs on
    the irreversible model, producing ras_<c1>_vs_<c2>.csv files used in the concordance
    analysis.
  repo: github.com/QLSLab/integrate
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1009337
    title: 'INTEGRATE: Model-based multi-omics data integration to characterize multi-level
      metabolic regulation'
  source_content_hash: sha256:bfb1381b53d21eda139e2b539fb8be2621b258c61d897ab7ab62e7b2fa12c1ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - INTEGRATE
  - Python
  - concordanceAnalysis
  - createMetabolicDataset
  - mannWhitneyUTest
  - rasTtest
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6551724137931034
n_claims: 29
---

# multi-omics-regulation-classification

## Summary

Classifies each metabolic reaction as regulated at the metabolic level, the transcriptional level, both, or neither, by intersecting RPS-vs-RAS and RPS-vs-FFD concordance scores computed from metabolomics and transcriptomics data on a constraint-based model.

## When to use

Use when you have paired transcriptomics and intracellular metabolomics data mapped onto a stoichiometric model and want to determine, per reaction, whether flux variation follows substrate abundance (metabolic), enzyme expression (transcriptional), both, or neither.

## When NOT to use

- The omics data are already integrated and only a single regulation layer is of interest — the intersection step adds no value.
- No constraint-based model covering the measured reactions is available; concordance analysis requires reaction-level GPRs and flux sampling on the model.
- Only extracellular metabolomics is measured; the paper's mass-action RPS derivation requires intracellular substrate abundances.

## Inputs

- Transcriptomics dataset (e.g. FPKM_Breast_forMarea.tsv)
- Intracellular metabolomics dataset (e.g. metabolomics_LM.csv)
- Conversion file between metabolomics IDs and model metabolite IDs (metsEngroVsMetabolomics.csv)
- Stoichiometric metabolic model in SBML format (e.g. ENGRO2_irrev.xml)
- Extracellular flux ratio dataset (ysi_ratio.csv)
- Nutrients availability dataset (medium.csv)

## Outputs

- Per-reaction concordance dataset with Cohen's kappa (linear-weighted) and Pearson correlation for RPS-vs-RAS and RPS-vs-FFD, plus p-values and adjusted p-values
- Heatmap of RPS-vs-RAS and RPS-vs-FFD concordance restricted to reactions with RPS-vs-FFD concordance > 0.2
- Scatterplot of RPS-vs-FFD (x) against RPS-vs-RAS (y) for reactions with all substrate abundances quantified
- Per-reaction regulatory class labels: metabolic-only, transcriptional+metabolic, transcriptional-only, or other

## How to apply

Run the full INTEGRATE chain: derive GPRs from the model (getGPRsFromModel), compute and normalize RAS scores from the transcriptomics file (getRASscore, getNormalizedRAS), integrate RAS plus extracellular-flux-ratio and nutrient-medium constraints into per-cell-line SBML models (rasIntegration with imposeYSI='Y', imposeMedium='Y', imposeRasConstraints='Y'), convert them to irreversible format, sample the feasible flux region (randomSampling), and compute Mann-Whitney U tests on the sampled fluxes to obtain flux fold-difference (FFD) significance per cell-line pair (mannWhitneyUTest). In parallel, run pairwise t-tests on RAS (rasTtest) and build the metabolomic statistical-test dataset with quality filter 1 and log-ratio threshold valLog=1.2 (createMetabolicDataset). Then execute concordanceAnalysis with linear-weighted Cohen's kappa to compute, per reaction, RPS-vs-RAS and RPS-vs-FFD concordance, Pearson correlation, p-values and adjusted p-values; reactions whose RPS-vs-FFD concordance exceeds 0.2 are carried forward. Finally, intersect the two concordance patterns to label each reaction as metabolic-only, transcriptional+metabolic, transcriptional-only, or other, mirroring the paper's Figure 4 and supplementary S4/S5 tables.

## Related tools

- **INTEGRATE** (Full pipeline implementing GPR extraction, RAS computation and normalization, RAS/medium/YSI integration into per-cell-line models, flux sampling, Mann-Whitney U FFD tests, RAS t-tests, metabolomic statistical-test construction, and concordance analysis with linear-weighted Cohen's kappa.) — github.com/QLSLab/integrate
- **concordanceAnalysis** (Computes per-reaction RPS-vs-RAS and RPS-vs-FFD Cohen's kappa (linear weighting) and Pearson correlation, p-values and adjusted p-values, and renders the heatmap and scatterplot used for regulation classification.) — github.com/QLSLab/integrate
- **mannWhitneyUTest** (Computes Mann-Whitney U tests on randomly sampled steady-state flux solutions per cell-line pair to obtain FFD significance feeding the concordance analysis.) — github.com/QLSLab/integrate
- **createMetabolicDataset** (Builds the metabolomic statistical-test dataset (quality filter 1, valLog=1.2) and per-pair log2 mean ratios used in the concordance analysis.) — github.com/QLSLab/integrate
- **rasTtest** (Runs pairwise t-tests on RAS scores for each of the 10 cell-line pairs on the irreversible model, producing ras_<c1>_vs_<c2>.csv files used in the concordance analysis.) — github.com/QLSLab/integrate

## Examples

```
python pipeline/concordanceAnalysis.py --valLog 1.2 --weight linear --resultsMetabolomicFile resultsMetabolomic --metabolic_model ENGRO2_irrev.xml --lcellLines ["MCF102A","SKBR3","MCF7","MDAMB231","MDAMB361"] --meansFile medie_Met.csv
```

## Evaluation signals

- concordance_scores.csv contains Cohen's kappa and Pearson values with p-values and adjusted p-values for RPS-vs-RAS and RPS-vs-FFD for every reaction
- The heatmap includes only reactions whose RPS-vs-FFD concordance exceeds 0.2, matching Figure 4 and the S4/S5 supplementary concordance tables
- Reactions ACONT, ASPTA (forward), RPI, and RPE show statistically significant RPS-vs-FFD concordance after Benjamini-Hochberg FDR correction
- Each reaction carries exactly one of four class labels: metabolic-only, transcriptional+metabolic, transcriptional-only, or other
- Per-pair outputs ras_<c1>_vs_<c2>.csv (RAS t-test) and FFD Mann-Whitney files exist for all 10 cell-line pairs of the 5-cell-line set

## Limitations

- Concordance analysis depends on the assumed relationship that a monotonic RPS-vs-FFD pattern indicates metabolic control and a non-monotonic RPS-vs-RAS pattern indicates transcriptional control; the classification is only as valid as these assumptions.
- The pipeline was tested using GLPK solver and MATLAB for reversible-to-irreversible model conversion; other solvers or converters are untested in the paper.
- Reactions whose substrate abundances are not all quantified cannot be rendered in the RPS-vs-FFD vs RPS-vs-RAS scatterplot.

## Evidence

- [methods] concordanceAnalysis usage and outputs: "Usage: `python pipeline/concordanceAnalysis.py`"
- [abstract] INTEGRATE intersection of the two output datasets: "We discriminate fluxes regulated at the metabolic and/or gene expression level by intersecting these two output datasets"
- [intro] Hypothesis underlying concordance interpretation: "Our novel hypothesis is that evidence for a monotonic relationship between variations in fluxes and variations in substrate abundances, and for a concurrent non-monotonic relationship between flux"
- [readme] Concordance analysis README description: "Aim: create the results (dataset and figures) for concordance analysis"
- [readme] Kappa weighting parameter in README: "weight: weight on the Kappa Cohen. Default value: 'linear'"
