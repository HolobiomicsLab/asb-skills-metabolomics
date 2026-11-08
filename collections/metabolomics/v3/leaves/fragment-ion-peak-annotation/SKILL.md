---
name: fragment-ion-peak-annotation
description: Use when you have validated peptide-spectrum matches (with known peptide
  sequences) plus de novo tool predictions and need to quantify missing fragment ions,
  noise peak fractions, or how these spectrum properties drive peptide recall.
when_to_use_negative:
- Spectra lack validated reference peptide sequences, so expected cleavage sites and
  the eight theoretical ion types cannot be enumerated.
- High-resolution data where a 0.5 Da tolerance is inappropriate or where the analysis
  goal is database-search PSM scoring rather than de novo error analysis.
- The goal is assembling full-length antibody chains (use the ALPS de Bruijn assembly
  skill instead of peak-level error annotation).
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Pyteomics
  role: Computes theoretical fragment ions for the eight ion types and matches observed
    peaks within the 0.5 Da tolerance to label cleavage sites as found or missing
- name: Casanovo
  role: De novo predictions whose peptide recall is stratified by missing cleavage
    sites, noise factor and peptide length
- name: PointNovo
  role: De novo predictions whose peptide recall is stratified by missing cleavage
    sites, noise factor and peptide length
- name: Novor
  role: De novo predictions whose peptide recall is stratified by missing cleavage
    sites, noise factor and peptide length
- name: pNovo 3
  role: De novo predictions whose peptide recall is stratified by missing cleavage
    sites, noise factor and peptide length
- name: DeepNovo
  role: De novo predictions whose peptide recall is stratified by missing cleavage
    sites, noise factor and peptide length
- name: SMSNet
  role: De novo predictions whose peptide recall is stratified by missing cleavage
    sites, noise factor and peptide length
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:82fbefa46b320c6c536edbc8dd2db7e47d22ffa5c8404b8eac52d5ca33a2bc33
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Casanovo
  - DeepNovo
  - Novor
  - PointNovo
  - Pyteomics
  - SMSNet
  - pNovo 3
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# fragment-ion-peak-annotation

## Summary

Annotates every peak in validated antibody MS/MS spectra as either a theoretical peptide fragment ion or a noise peak, then derives spectrum-level statistics (missing cleavage sites, noise factor) that explain de novo sequencing recall. It operationalizes the Beslic et al. 2022 error-type analysis of Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo predictions.

## When to use

Use when you have validated peptide-spectrum matches (with known peptide sequences) plus de novo tool predictions and need to quantify missing fragment ions, noise peak fractions, or how these spectrum properties drive peptide recall.

## When NOT to use

- Spectra lack validated reference peptide sequences, so expected cleavage sites and the eight theoretical ion types cannot be enumerated.
- High-resolution data where a 0.5 Da tolerance is inappropriate or where the analysis goal is database-search PSM scoring rather than de novo error analysis.
- The goal is assembling full-length antibody chains (use the ALPS de Bruijn assembly skill instead of peak-level error annotation).

## Inputs

- Validated MS/MS spectra with known peptide sequences (23,227 antibody spectra across three multi-enzymatic digest data sets)
- De novo peptide predictions from Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo (from the benchmarking task)
- Peak lists per spectrum with m/z and intensity values (e.g. from mzML/MGF)

## Outputs

- Per-spectrum statistics table (spectrum_statistics.csv): missing cleavage sites, noise peak fraction, noise factor
- Recall-vs-missing-cleavages curves per tool (Figure 3A)
- Recall-vs-noise-factor curves per tool (Figure 3B)
- Peptide-length x missing-cleavage-sites recall heatmap (Figure 4)

## How to apply

Load the validated spectra and the de novo predictions for all six tools (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) from the prior benchmarking step. Using Pyteomics, for every expected cleavage site of the known peptide attempt to match eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) within a 0.5 Da mass tolerance, and declare a cleavage site missing when no observed peak matches. Classify as noise peaks only those peaks exceeding the median noise intensity of the data set, and compute a per-spectrum noise factor as the ratio of high-intensity noise peaks to fragment ion peaks. Bin spectra by number of missing cleavage sites and by noise factor to compute per-tool peptide recall curves, and cross-tabulate recall by peptide length and missing cleavages for the accuracy matrix. The rationale is that missing fragmentation sites, noisy spectra and long peptide sequences are the dominant failure modes of all six de novo tools, so peak-level annotation turns raw spectra into interpretable covariates of recall.

## Related tools

- **Pyteomics** (Computes theoretical fragment ions for the eight ion types and matches observed peaks within the 0.5 Da tolerance to label cleavage sites as found or missing)
- **Casanovo** (De novo predictions whose peptide recall is stratified by missing cleavage sites, noise factor and peptide length)
- **PointNovo** (De novo predictions whose peptide recall is stratified by missing cleavage sites, noise factor and peptide length)
- **Novor** (De novo predictions whose peptide recall is stratified by missing cleavage sites, noise factor and peptide length)
- **pNovo 3** (De novo predictions whose peptide recall is stratified by missing cleavage sites, noise factor and peptide length)
- **DeepNovo** (De novo predictions whose peptide recall is stratified by missing cleavage sites, noise factor and peptide length)
- **SMSNet** (De novo predictions whose peptide recall is stratified by missing cleavage sites, noise factor and peptide length)

## Examples

```
from pyteomics import mass; IONS=['b','b(2+)','b-NH3','b-H2O','y','y(2+)','y-NH3','y-H2O']; hits=[p for p in peaks if min(abs(p.mz-mass.fast_mass2(seq,ion=i,charge=c)))<0.5]
```

## Evaluation signals

- Reproduced aggregate statistics must match the paper's reported values: 90.51% of the 23,227 validated spectra missing at least one fragment ion and 84.32% of peaks classified as noise
- Noise peak set must be restricted to peaks exceeding the median noise intensity per data set, not all non-matching peaks
- Recall curves must show Casanovo with the highest peptide recall on spectra with up to eight missing cleavage sites
- Noise-factor bins should show little effect of noise factor alone on peptide recall, consistent with the paper
- Every annotated spectrum should carry counts of missing cleavage sites over the eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O)

## Limitations

- The 0.5 Da tolerance and eight-ion-type matching scheme assume the instrument resolution of the antibody data sets; isoleucine/leucine and other isobaric ambiguities cannot be resolved by MS/MS peak annotation alone
- Noise peaks are only counted when they exceed the median noise intensity per data set, so low-intensity chemical or electronic noise is ignored
- Missing fragmentation sites, noisy spectra and long peptide sequences pose a limit for all de novo sequencing tools, so spectra hitting these limits yield poor recall regardless of the tool
- Performance of DL-based tools on non-tryptic data varies with the cleavage pattern of the trained data set, which conditions the recall stratification

## Evidence

- [intro] we investigate common error types, the impact of noisy spectra and missing fragmentation ions: "Furthermore, we investigate common error types, the impact of noisy spectra and missing fragmentation ions."
- [discussion] The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools: "The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools"
- [methods] for every cleavage site attempt to match eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) within a 0.5 Da tolerance: [evidence span withheld — anchor status: not_found]
- [results] 90.51% of the 23,227 validated spectra were missing at least one fragment ion, 84.32% of peaks in these spectra were noise: [evidence span withheld — anchor status: not_found]
