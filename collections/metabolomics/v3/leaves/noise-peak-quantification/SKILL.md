---
name: noise-peak-quantification
description: Use when you have validated MS/MS spectra with known ground-truth peptides
  plus de novo predictions, and need to characterize noise levels, missing cleavage-site
  ions, or their effect on peptide recall.
when_to_use_negative:
- Spectra lack validated ground-truth peptide sequences, so per-spectrum peptide recall
  cannot be computed
- The goal is absolute noise characterization of raw unsearched spectra rather than
  relating spectrum quality to de novo tool performance
- Data are not enzymatic antibody digest MS/MS data sets evaluated with the six de
  novo sequencing tools
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: Pyteomics
  role: Annotates peaks against the eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+),
    y-NH3, y-H2O) at 0.5 Da tolerance to classify peptide vs noise peaks and detect
    missing cleavage sites
- name: Casanovo
  role: Deep learning de novo sequencer whose peptide recall is computed per spectrum-quality
    bin; performs best on spectra with up to eight missing cleavage sites
- name: PointNovo
  role: Deep learning de novo sequencer whose peptide recall is computed per missing-cleavage-site
    and noise-factor bin for comparison
- name: Novor
  role: Spectrum-graph-based de novo sequencer included in the recall-by-spectrum-quality
    comparison
- name: pNovo 3
  role: Spectrum-graph-based de novo sequencer included in the recall-by-spectrum-quality
    comparison
- name: DeepNovo
  role: Deep learning de novo sequencer included in the recall-by-spectrum-quality
    comparison
- name: SMSNet
  role: Deep learning de novo sequencer included in the recall-by-spectrum-quality
    comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:c8214e8a867b28d4aac6ead0306e1fb3b5614861c894364b5a389cd5cd0a19d2
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# noise-peak-quantification

## Summary

Quantifies missing fragment ions and noise peaks in validated antibody MS/MS spectra and measures their impact on peptide recall of de novo sequencing tools (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo). It explains which spectra-quality features limit de novo antibody sequencing accuracy.

## When to use

Use when you have validated MS/MS spectra with known ground-truth peptides plus de novo predictions, and need to characterize noise levels, missing cleavage-site ions, or their effect on peptide recall.

## When NOT to use

- Spectra lack validated ground-truth peptide sequences, so per-spectrum peptide recall cannot be computed
- The goal is absolute noise characterization of raw unsearched spectra rather than relating spectrum quality to de novo tool performance
- Data are not enzymatic antibody digest MS/MS data sets evaluated with the six de novo sequencing tools

## Inputs

- Validated antibody MS/MS spectra with ground-truth peptide assignments (23,227 spectra across three multi-enzymatic data sets)
- De novo peptide predictions from Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo
- Pyteomics for peak annotation and ion-type mass calculation

## Outputs

- spectrum_statistics.csv with spectrum-level missing-fragment-ion counts, noise peak fractions and noise factors
- Recall-vs-missing-cleavages and recall-vs-noise-factor curves (Figure 3A/3B)
- Peptide-length × number-of-missing-cleavages recall heatmap (Figure 4)

## How to apply

Load validated spectra and de novo predictions for all six tools (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) from the antibody data sets. Using Pyteomics, label each peak as peptide peak or noise peak: for every cleavage site attempt to match eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) within a 0.5 Da tolerance, declaring a cleavage site missing when no ion matches. Count as noise peaks only those exceeding the median noise intensity per data set, and compute the noise factor per spectrum as the ratio of high-intensity noise peaks to fragment ion peaks. Bin validated spectra by number of missing cleavage sites and by noise factor, then compute peptide recall per bin per tool to build the Figure 3A/3B curves and the length × missing-cleavage accuracy matrix (Figure 4). Write spectrum-level statistics to spectrum_statistics.csv.

## Related tools

- **Pyteomics** (Annotates peaks against the eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) at 0.5 Da tolerance to classify peptide vs noise peaks and detect missing cleavage sites)
- **Casanovo** (Deep learning de novo sequencer whose peptide recall is computed per spectrum-quality bin; performs best on spectra with up to eight missing cleavage sites)
- **PointNovo** (Deep learning de novo sequencer whose peptide recall is computed per missing-cleavage-site and noise-factor bin for comparison)
- **Novor** (Spectrum-graph-based de novo sequencer included in the recall-by-spectrum-quality comparison)
- **pNovo 3** (Spectrum-graph-based de novo sequencer included in the recall-by-spectrum-quality comparison)
- **DeepNovo** (Deep learning de novo sequencer included in the recall-by-spectrum-quality comparison)
- **SMSNet** (Deep learning de novo sequencer included in the recall-by-spectrum-quality comparison)

## Examples

```
from pyteomics import mass; mz_b = mass.fast_mass(seq[:k], ion_type='b'); missing = not any(abs(p - mz_b) <= 0.5 for p in peaks)
```

## Evaluation signals

- Aggregate statistics must match the reported values: 90.51% of the 23,227 validated spectra missing at least one fragment ion and 84.32% of peaks in these spectra classified as noise
- Noise peaks counted only above the median noise intensity per data set; noise factor equals ratio of high-intensity noise peaks to fragment ion peaks per spectrum
- Recall-vs-missing-cleavages curves must show Casanovo with the highest peptide recall on spectra with up to eight missing cleavage sites
- Recall-vs-noise-factor curves must show that noise factor alone has little effect on peptide recall
- Cleavage sites declared missing only when none of the eight ion types matches within 0.5 Da

## Limitations

- The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools
- Discrimination of isoleucine and leucine cannot be achieved via MS/MS and requires MS3 fragmentation, so recall metrics cannot resolve such isomeric residues
- Particular amino acids (e.g. I & L; Q & AG; deamidated N & D) cannot be resolved even with higher-resolution spectra and tools
- Performance of DL tools on non-tryptic data varies based on the cleavage pattern of the trained data set

## Evidence

- [intro] Furthermore, we investigate common error types, the impact of noisy spectra and missing fragmentation ions.: "Furthermore, we investigate common error types, the impact of noisy spectra and missing fragmentation ions."
- [discussion] The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools: "The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools"
- [other] 90.51% of the 23,227 validated spectra were missing at least one fragment ion, 84.32% of peaks in these spectra were noise: [evidence span withheld — anchor status: not_found]
- [other] for every cleavage site attempt to match eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) within a 0.5 Da tolerance: [evidence span withheld — anchor status: not_found]
