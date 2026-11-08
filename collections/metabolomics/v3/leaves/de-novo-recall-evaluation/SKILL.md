---
name: de-novo-recall-evaluation
description: Use when de novo predictions (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo,
  Casanovo) exist for validated antibody spectra and you must explain recall variation
  by missing fragment ions, missing cleavage sites, noise factor, or peptide length.
when_to_use_negative:
- The de novo predictions lack a validated ground-truth peptide set, so peptide recall
  per bin cannot be computed
- The goal is assembling full-length antibody sequences (e.g. with the de Bruijn assembler
  ALPS) rather than diagnosing per-spectrum recall degradation
- You need precision-oriented filtering (e.g. precursor mass filtering of predictions)
  rather than recall-vs-spectrum-quality analysis
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pyteomics
  role: 'Peak labeling: matching eight ion types per cleavage site within 0.5 Da tolerance
    and classifying noise peaks against the median noise intensity'
- name: Casanovo
  role: De novo predictions evaluated for recall across missing-cleavage-site and
    noise-factor bins
- name: PointNovo
  role: De novo predictions evaluated for recall across missing-cleavage-site and
    noise-factor bins
- name: Novor
  role: De novo predictions evaluated for recall as spectrum-graph-based baseline
- name: pNovo 3
  role: De novo predictions evaluated for recall as spectrum-graph-based baseline
- name: DeepNovo
  role: De novo predictions evaluated for recall as DL-based baseline
- name: SMSNet
  role: De novo predictions evaluated for recall as DL-based baseline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbac542
    title: Comprehensive evaluation of peptide <i>de novo</i> sequencing tools for
      monoclonal antibody assembly
  source_content_hash: sha256:0d3dbffafce957c05f357d575ea7c18d49c95bbd12ef3872a3dbbdbefb2167b5
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# de-novo-recall-evaluation

## Summary

Quantifies how missing fragment ions, missing cleavage sites, noise peaks, and peptide length degrade peptide recall of de novo sequencing tools on validated antibody MS/MS spectra, using Pyteomics-based peak labeling into ion-type vs noise classes.

## When to use

Use when de novo predictions (Novor, pNovo 3, DeepNovo, SMSNet, PointNovo, Casanovo) exist for validated antibody spectra and you must explain recall variation by missing fragment ions, missing cleavage sites, noise factor, or peptide length.

## When NOT to use

- The de novo predictions lack a validated ground-truth peptide set, so peptide recall per bin cannot be computed
- The goal is assembling full-length antibody sequences (e.g. with the de Bruijn assembler ALPS) rather than diagnosing per-spectrum recall degradation
- You need precision-oriented filtering (e.g. precursor mass filtering of predictions) rather than recall-vs-spectrum-quality analysis

## Inputs

- Validated antibody MS/MS spectra (23,227 spectra from the three multi-enzymatic antibody data sets)
- De novo peptide predictions from Novor, pNovo 3, DeepNovo, SMSNet, PointNovo and Casanovo
- Pyteomics (Python) for peak labeling and ion-type matching

## Outputs

- spectrum_statistics.csv (per-spectrum missing-cleavage-site counts, noise peak fractions, noise factors)
- figure_3.png (peptide recall vs. number of missing cleavage sites and vs. noise factor, per tool)
- figure_4.png (peptide recall heatmap: peptide length x number of missing cleavage sites)

## How to apply

Load the validated spectra and the de novo predictions for the six tools from the prior sequencing task. Using Pyteomics, label every peak as a peptide peak or noise peak: for each cleavage site, attempt to match eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) within a 0.5 Da tolerance, declaring the cleavage site missing when no ion matches; count as noise peaks only those exceeding the median noise intensity per data set. Compute the noise factor per spectrum as the ratio of high-intensity noise peaks to fragment ion peaks, and verify aggregate statistics against the paper's reported values (90.51% of the 23,227 validated spectra missing at least one fragment ion; 84.32% of peaks classified as noise). Bin validated spectra by number of missing cleavage sites and by noise factor, then compute peptide recall per bin per tool to build the recall curves. Cross-tabulate recall by peptide length and number of missing cleavage sites to build the accuracy heatmap. The decision point is interpretation: expect DL-based tools (Casanovo, PointNovo) to lead recall, with Casanovo best on spectra with up to eight missing cleavage sites, while noise factor alone has little effect.

## Related tools

- **Pyteomics** (Peak labeling: matching eight ion types per cleavage site within 0.5 Da tolerance and classifying noise peaks against the median noise intensity)
- **Casanovo** (De novo predictions evaluated for recall across missing-cleavage-site and noise-factor bins)
- **PointNovo** (De novo predictions evaluated for recall across missing-cleavage-site and noise-factor bins)
- **Novor** (De novo predictions evaluated for recall as spectrum-graph-based baseline)
- **pNovo 3** (De novo predictions evaluated for recall as spectrum-graph-based baseline)
- **DeepNovo** (De novo predictions evaluated for recall as DL-based baseline)
- **SMSNet** (De novo predictions evaluated for recall as DL-based baseline)

## Examples

```
from pyteomics import mzml; spectra = mzml.read('validated_spectra.mzML'); ions = ['b','b2','b-NH3','b-H2O','y','y2','y-NH3','y-H2O']  # match each cleavage site within 0.5 Da tolerance
```

## Evaluation signals

- Reproduced aggregate statistics must match the paper: 90.51% of the 23,227 validated spectra are missing at least one fragment ion, and 84.32% of peaks in these spectra are noise
- Recall-vs-missing-cleavages curves must show Casanovo with the highest peptide recall on spectra with up to eight missing cleavage sites
- Recall-vs-noise-factor curves must show noise factor alone has little effect on peptide recall
- Per-bin recall values must be non-decreasing in expected direction: tools identify more correct peptides from spectra with fewer missing fragment ions
- The Figure 4 output must be a valid cross-tabulation (peptide length x missing cleavage sites) with recall as the cell value for each of the six tools

## Limitations

- The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools — recall degrades in all bins beyond these thresholds
- Noise classification is restricted to peaks exceeding the median noise intensity per data set; noise factor alone has little effect on recall, so it is a weak explanatory variable
- DL tool performance on non-tryptic enzymatic digests varies based on the cleavage pattern of the training data, which confounds cross-digest recall comparisons

## Evidence

- [results] 90.51% of the 23,227 validated spectra were missing at least one fragment ion, 84.32% of peaks in these spectra were noise: [evidence span withheld — anchor status: not_found]
- [methods] for every cleavage site attempt to match eight ion types (b, b(2+), b-NH3, b-H2O, y, y(2+), y-NH3, y-H2O) within a 0.5 Da tolerance: [evidence span withheld — anchor status: not_found]
- [discussion] The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools: "The number of missing fragmentation sites, noisy spectra and long peptide sequences poses a limit for all de novo sequencing tools"
- [abstract] The deep learning-based tools Casanovo and PointNovo showed an increased peptide recall across different enzymes and data sets compared with spectrum-graph-based: "The deep learning-based tools Casanovo and PointNovo showed an increased peptide recall across different enzymes and data sets compared with spectrum-graph-based approaches."
- [intro] Furthermore, we investigate common error types, the impact of noisy spectra and missing fragmentation ions.: "Furthermore, we investigate common error types, the impact of noisy spectra and missing fragmentation ions."
