---
name: peptide-length-accuracy-analysis
description: Use when de novo peptide predictions for antibody MS/MS spectra have
  been validated and you need to quantify whether long peptides are predicted less
  accurately and how missing fragment ions amplify that loss.
when_to_use_negative:
- Input spectra have no validated reference peptide sequences, so recall per length
  cannot be computed
- De novo predictions have not yet been generated or aligned to spectra — run the
  sequencing skill first
- The goal is database-driven peptide identification, not de novo recall benchmarking
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Pyteomics
  role: Match fragment ion types (b, y, NH3/H2O losses, 2+ charge states) within 0.5
    Da tolerance to determine which cleavage sites are missing, enabling the length
    x missing-cleavage stratification
- name: Casanovo
  role: One of the six evaluated de novo sequencing tools whose recall is stratified
    by peptide length and missing cleavage sites; performs best on spectra with up
    to eight missing cleavage sites
- name: PointNovo
  role: Deep learning-based tool whose recall is stratified by peptide length and
    missing cleavage sites; showed increased peptide recall across enzymes and data
    sets compared with spectrum-graph-based approaches
- name: DeepNovo
  role: Deep learning-based de novo sequencing tool whose recall is stratified by
    peptide length in the Figure 4 accuracy matrix
- name: SMSNet
  role: Deep learning-based de novo sequencing tool whose recall is stratified by
    peptide length in the Figure 4 accuracy matrix
- name: Novor
  role: Spectrum-graph-based de novo sequencing tool whose recall is stratified by
    peptide length in the Figure 4 accuracy matrix
- name: pNovo 3
  role: Spectrum-graph-based de novo sequencing tool whose recall is stratified by
    peptide length in the Figure 4 accuracy matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:8d61f684c50e093059ae4c754961f242d9f22e14e68f942f8cd493358d6e6a5e
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# peptide-length-accuracy-analysis

## Summary

Cross-tabulates peptide recall of de novo sequencing predictions by peptide length, revealing how spectral quality degradation such as missing cleavage sites compounds the length-driven drop in accuracy.

## When to use

Use when de novo peptide predictions for antibody MS/MS spectra have been validated and you need to quantify whether long peptides are predicted less accurately and how missing fragment ions amplify that loss.

## When NOT to use

- Input spectra have no validated reference peptide sequences, so recall per length cannot be computed
- De novo predictions have not yet been generated or aligned to spectra — run the sequencing skill first
- The goal is database-driven peptide identification, not de novo recall benchmarking

## Inputs

- Validated monoclonal antibody MS/MS spectra with reference peptide identities (23,227 spectra across multi-enzymatic digests)
- De novo peptide prediction files for Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo
- Per-spectrum missing cleavage site counts and noise factor statistics from spectrum_statistics.csv

## Outputs

- Peptide recall matrix (tools x peptide length x missing cleavage sites) as figure_4.png heatmap
- Per-cell recall values written alongside spectrum_statistics.csv for downstream comparison

## How to apply

Take the per-spectrum validated peptide identities from task_001 and the de novo predictions for each tool (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo). Join each predicted peptide to its validated spectrum and bin spectra by validated peptide length (amino acid count) and by the number of missing cleavage sites determined with Pyteomics, where a cleavage site is declared missing when none of the eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) matches within 0.5 Da. Compute peptide recall per length x missing-cleavage-site cell for each tool and render the matrix as a heatmap (Figure 4 in the paper). Rationale: the article reports that long peptide sequences are a limit for all tools, so length-stratified recall isolates where each algorithm fails and whether spectra with more missing fragment ions show disproportionately lower recall at long lengths.

## Related tools

- **Pyteomics** (Match fragment ion types (b, y, NH3/H2O losses, 2+ charge states) within 0.5 Da tolerance to determine which cleavage sites are missing, enabling the length x missing-cleavage stratification)
- **Casanovo** (One of the six evaluated de novo sequencing tools whose recall is stratified by peptide length and missing cleavage sites; performs best on spectra with up to eight missing cleavage sites)
- **PointNovo** (Deep learning-based tool whose recall is stratified by peptide length and missing cleavage sites; showed increased peptide recall across enzymes and data sets compared with spectrum-graph-based approaches)
- **DeepNovo** (Deep learning-based de novo sequencing tool whose recall is stratified by peptide length in the Figure 4 accuracy matrix)
- **SMSNet** (Deep learning-based de novo sequencing tool whose recall is stratified by peptide length in the Figure 4 accuracy matrix)
- **Novor** (Spectrum-graph-based de novo sequencing tool whose recall is stratified by peptide length in the Figure 4 accuracy matrix)
- **pNovo 3** (Spectrum-graph-based de novo sequencing tool whose recall is stratified by peptide length in the Figure 4 accuracy matrix)

## Examples

```
python compute_recall_by_length.py --validated validated_spectra.csv --predictions casanovo.csv --ions b,y,b2,y2 --tol 0.5 --out figure_4.png
```

## Evaluation signals

- The number of missing cleavage sites and peptide lengths span the range seen across the 23,227 validated spectra; no tool cell should have zero denominator unless the length bin is empty
- Reproducing the paper's Figure 4 should show peptide recall decreasing with increasing peptide length for all six tools
- Casanovo should retain the highest recall across bins with up to eight missing cleavage sites, consistent with the paper's reported result
- The computed noise/fragment statistics must match the paper's reported 90.51% of spectra with at least one missing fragment ion and 84.32% noise peaks
- Missing cleavage site counts derived from the 0.5 Da ion-match rule must be reproducible: rerunning the Pyteomics annotation on the same spectra yields identical per-spectrum counts

## Limitations

- Missing fragmentation sites, noisy spectra and long peptide sequences jointly pose a limit for all de novo sequencing tools
- Particular amino acids cannot be resolved even with higher-resolution spectra and tools (e.g. I & L; Q & AG; deamidated N & D), which inflates apparent length-related errors
- Discrimination of the isomeric residues isoleucine and leucine cannot be achieved via MS/MS and requires MS3 fragmentation
- Performance of deep learning tools on non-tryptic data varies based on the cleavage pattern of the trained data set

## Evidence

- [discussion] The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools: "The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools"
- [other] 90.51% of the 23,227 validated spectra were missing at least one fragment ion: [evidence span withheld — anchor status: not_found]
- [discussion] DL tools performance varies with trained cleavage pattern: "DL tools can still be applied to different enzymatic data sets, although the performance will vary based on the cleavage pattern of the trained data set"
- [discussion] Unresolvable isobaric residues affect length-based accuracy: "However, particular amino acids cannot be resolved even with spectra and tools with higher resolution (e.g. I & L; Q & AG; deamidated N & D)"
