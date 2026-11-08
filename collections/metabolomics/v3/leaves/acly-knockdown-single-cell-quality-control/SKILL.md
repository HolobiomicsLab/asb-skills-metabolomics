---
name: acly-knockdown-single-cell-quality-control
description: Use when analysing single-cell anndata objects from the 13C-SpaceM ACLYkd
  slide 20220411_AB_DKFZACLYac (control vs ACLYkd oligo 1/2 wells) and you must QC
  cells, build pseudo-bulk palmitate isotopologue distributions, or benchmark pseudo-bulk
  M+0 fractions against bulk LC-MS values.
when_to_use_negative:
- Hypoxia/normoxia 13C-SpaceM data where overall substrate usage is too low to reliably
  fit a binomial distribution — the article states the binomial fit was only used
  for ACLYkd data
- Mouse brain tissue data, where the equivalent analysis is done per MALDI-image pixel
  rather than per segmented cell
- Raw imzML preprocessing/extraction of isotopologue ion images — the README shares
  SpaceM output so this step need not be rerun
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: 13C-SpaceM
  role: Repository and per-slide workflow (sc_analysis.py) generating natural-abundance-corrected,
    normalized single-cell isotopologue anndata for the ACLYkd slides
  repo: https://github.com/Buglakova/13C-SpaceM
- name: 13C-binomial
  role: fit_binomial.py fits the binomial model (C16/C18) to corrected pseudo-bulk
    palmitate isotopologue intensities, yielding uptake, p, mean and fit_success
  repo: https://github.com/Buglakova/13C_binomial
- name: Scanpy
  role: Stores and reads the pickled single-cell anndata objects that carry isotopologue
    fractions per cell before QC filtering
- name: SpaceM
  role: Closed-source upstream software whose shared output (microscopy–MALDI registration,
    segmentation, overlap analysis) is the input to the per-slide single-cell analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:3fc566c309a689cd900099c6bdb271543176f9127e6d1ecd3ea6cf763ad8c1fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Buglakova/13C-SpaceM
  tools:
  - 13C-SpaceM
  - 13C-binomial
  - Scanpy
  - SpaceM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# acly-knockdown-single-cell-quality-control

## Summary

Quality-control and pseudo-bulk aggregation workflow for 13C-SpaceM single-cell isotope-tracing data from the ACLY knockdown (ACLYkd) experiment, filtering cells by size, ion count and total ion intensity before comparing palmitate isotopologue distributions against bulk LC-MS measurements.

## When to use

Use when analysing single-cell anndata objects from the 13C-SpaceM ACLYkd slide 20220411_AB_DKFZACLYac (control vs ACLYkd oligo 1/2 wells) and you must QC cells, build pseudo-bulk palmitate isotopologue distributions, or benchmark pseudo-bulk M+0 fractions against bulk LC-MS values.

## When NOT to use

- Hypoxia/normoxia 13C-SpaceM data where overall substrate usage is too low to reliably fit a binomial distribution — the article states the binomial fit was only used for ACLYkd data
- Mouse brain tissue data, where the equivalent analysis is done per MALDI-image pixel rather than per segmented cell
- Raw imzML preprocessing/extraction of isotopologue ion images — the README shares SpaceM output so this step need not be rerun

## Inputs

- Scanpy pickled anndata objects per slide/well produced by sciso/sc_analysis.py from 20220411_AB_DKFZACLYac SpaceM output
- SpaceM output directories (spatiomolecular_adata.h5ad, cell segmentation and overlap analysis files)
- Molecule/ion list CSV (e.g. analysis/ion_list_fa.csv) defining fatty acid isotopologues
- Bulk LC-MS isotopologue measurements for control cells (reference for pseudo-bulk M+0 comparison)

## Outputs

- QC-filtered, GFP-classified single-cell anndata object with corrected and normalized isotopologue fractions per cell
- palmitate_isotopologue_distributions_aclykd.csv (pseudo-bulk isotopologue fractions per condition)
- palmitate_isotopologue_shift_aclykd.png (ACLYkd vs control isotopologue-shift plot)
- fit_binomial_result.csv from 13C-binomial: model (C16), fit_success, uptake, p, mean, p_mean per sample

## How to apply

Load the Scanpy pickled anndata objects produced per slide by running `python sciso/sc_analysis.py` on the SpaceM output of slides 1–3 of 20220411_AB_DKFZACLYac; that script performs natural isotope abundance correction per fatty acid, median-aggregates signal per cell, and normalizes isotopologue distributions (M+0 … M+16) per cell. Aggregate slides, then apply the manuscript's quality control from the analysis/ACLYkd notebooks: filter cells by size, by number of detected ions, and by total ion intensity, and classify cells by normalized GFP intensity. Median-average corrected isotopologue fractions into pseudo-bulk distributions per condition (control, ACLYkd oligo 1, ACLYkd oligo 2) and fit the binomial model via Buglakova/13C_binomial's `fit_binomial` (model C16 for palmitate; numpy array of corrected, normalized intensities as input), extracting uptake and p parameters from fit_binomial_result.csv. Verify correctness by reproducing the shift toward lower-mass isotopologues in ACLYkd versus control and the control pseudo-bulk palmitate M+0 fraction of ~0.6 versus bulk LC-MS ~0.3; save palmitate_isotopologue_distributions_aclykd.csv and palmitate_isotopologue_shift_aclykd.png.

## Related tools

- **13C-SpaceM** (Repository and per-slide workflow (sc_analysis.py) generating natural-abundance-corrected, normalized single-cell isotopologue anndata for the ACLYkd slides) — https://github.com/Buglakova/13C-SpaceM
- **13C-binomial** (fit_binomial.py fits the binomial model (C16/C18) to corrected pseudo-bulk palmitate isotopologue intensities, yielding uptake, p, mean and fit_success) — https://github.com/Buglakova/13C_binomial
- **Scanpy** (Stores and reads the pickled single-cell anndata objects that carry isotopologue fractions per cell before QC filtering)
- **SpaceM** (Closed-source upstream software whose shared output (microscopy–MALDI registration, segmentation, overlap analysis) is the input to the per-slide single-cell analysis)

## Examples

```
python sciso/sc_analysis.py $data_dir/20220411_AB_DKFZACLYac/slide1/spacem_data $data_dir/20220411_AB_DKFZACLYac/slide1/anndata
```

## Evaluation signals

- After running sc_analysis.py on every slide, a single joined anndata object per slide (all wells) exists with finite normalized isotopologue fractions for palmitate (M+0 … M+16)
- QC removes cells below thresholds on size, number of detected ions and total ion intensity before pseudo-bulk aggregation; GFP intensity classification is present in the final anndata
- ACLYkd oligo 1 and oligo 2 pseudo-bulk palmitate distributions shift toward lower-mass isotopologues relative to control, indicating reduced glucose-derived carbon in the lipogenic acetyl-CoA pool
- Control pseudo-bulk palmitate M+0 fraction is ~0.6, reproducible against the bulk LC-MS value of ~0.3 discrepancy reported in the manuscript
- fit_binomial_result.csv for ACLYkd samples shows fit_success = 1 with model C16 and finite uptake/p parameters

## Limitations

- The binomial fit becomes unreliable when overall substrate usage is very low (as in the hypoxia/normoxia experiment), so it is only applied to the ACLYkd data
- Silencing ACLY caused reduced fractional labelling of acetyl-CoA from glucose with only a minor increase in the M+0 fraction of palmitate — expect a distribution shift rather than a dramatic M+0 change
- Substantial single-cell heterogeneity in fractional labelling within ACLY-silenced populations is obscured in bulk and reflects differences in shRNA oligo efficiency
- Scanpy pickled anndata can fail to load across mismatched pandas/anndata/pickle/scanpy versions; consult environment_dump.yml for troubleshooting
- Bulk measurements and pseudo-bulk single-cell M+0 fractions differ systematically in control cells (pseudo-bulk ~0.6 vs bulk ~0.3), so they are not interchangeable references

## Evidence

- [methods] After single-cell analysis data from all slides is aggregated together and quality control is performed, filtering out cells by size and number and total intensity of detected ions: "After single-cell analysis data from all slides is aggregated together and quality control is performed, filtering out cells by size and number and total intensity of detected ions"
- [discussion] ACLY discussion finding: "silencing of ACLY caused a reduction in the fractional labelling of acetyl-CoA from glucose but only a minor increase in the M + 0 fraction of palmitate."
- [discussion] Binomial fit unreliability at low substrate usage: "if overall substrate usage is very low, such as in the hypoxia/normoxia experiment, it becomes impossible to reliably fit a binomial distribution and estimate the uptake fraction"
- [readme] fit_binomial input description: [evidence span withheld — anchor status: external_doc]
- [readme] Scanpy anndata storage: "This project uses Scanpy's pickled `anndata` objects to store the single-cell data."
