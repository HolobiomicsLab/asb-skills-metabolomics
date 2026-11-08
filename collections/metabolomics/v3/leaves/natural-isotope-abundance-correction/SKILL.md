---
name: natural-isotope-abundance-correction
description: Use when isotopologue ion intensities of 13C6-glucose-labelled fatty
  acids (e.g. palmitate C16, stearate/oleate C18) are extracted per cell or per pixel
  from imzML-derived ion images and must be freed of natural-abundance 13C signal
  before binomial fitting.
when_to_use_negative:
- 'Before correction: no, correction applies only AFTER raw isotopologue intensities
  are extracted; applying it to SpaceM/METASPACE annotations is redundant because
  Metaspace only generates isotopologue distributions based on natural isotope abundance'
- When overall substrate usage is very low (e.g. hypoxia/normoxia experiment), correction
  alone cannot rescue downstream fitting — binomial modelling becomes unreliable and
  uptake estimation should be skipped
- When analyzing lipid classes poorly ionized in negative MALDI-AIF mode (phosphatidylcholines,
  triglycerides, sphingomyelins), since correction does not address the ionization
  selectivity limitation
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: pyimzML
  role: Parses imzML files to produce the raw per-pixel/per-ablation-mark isotopologue
    ion images that feed the correction step
  repo: https://github.com/alexandrovteam/pyimzML
- name: 13C-SpaceM
  role: Runs sciso/sc_analysis.py, which performs natural isotope abundance correction
    for each fatty acid in each ablation mark before median aggregation and normalization
  repo: https://github.com/Buglakova/13C-SpaceM
- name: 13C_binomial
  role: Consumes the corrected and normalized isotopologue intensities via fit_binomial
    (C16/C18 models); the correction is a prerequisite for its binomial fit
  repo: https://github.com/Buglakova/13C_binomial
- name: Scanpy
  role: Stores corrected single-cell isotopologue distributions in pickled anndata
    objects per slide
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_007
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:de01652a1eef68b300ff6b98d73efc530aa36982cb7a7d398f0e9c42c00e49d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/pyimzML
  tools:
  - 13C-SpaceM
  - 13C_binomial
  - METASPACE
  - Scanpy
  - SpaceM
  - conda
  - mamba
  - pyimzML
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# natural-isotope-abundance-correction

## Summary

Corrects fatty acid isotopologue distributions (M+0 ... M+n) measured by 13C-SpaceM single-cell MALDI imaging or MALDI-AIF tissue imaging to remove isotopologue signal expected from natural 13C abundance, so that downstream binomial modelling reflects glucose-derived labelling only.

## When to use

Use when isotopologue ion intensities of 13C6-glucose-labelled fatty acids (e.g. palmitate C16, stearate/oleate C18) are extracted per cell or per pixel from imzML-derived ion images and must be freed of natural-abundance 13C signal before binomial fitting.

## When NOT to use

- Before correction: no, correction applies only AFTER raw isotopologue intensities are extracted; applying it to SpaceM/METASPACE annotations is redundant because Metaspace only generates isotopologue distributions based on natural isotope abundance
- When overall substrate usage is very low (e.g. hypoxia/normoxia experiment), correction alone cannot rescue downstream fitting — binomial modelling becomes unreliable and uptake estimation should be skipped
- When analyzing lipid classes poorly ionized in negative MALDI-AIF mode (phosphatidylcholines, triglycerides, sphingomyelins), since correction does not address the ionization selectivity limitation

## Inputs

- Per-ablation-mark or per-pixel isotopologue ion intensities (M+0 to M+16) for fatty acids, extracted from imzML files with pyimzML
- Ion/molecule list CSV (e.g. analysis/ion_list_fa.csv or analysis/brain/ion_list_fa.csv) defining fatty acid isotopologues
- Scanpy pickled anndata objects with single-cell isotopologue data for the sc_analysis.py route

## Outputs

- Natural-isotope-abundance-corrected isotopologue distributions per fatty acid per cell (or pixel)
- Normalized fractional isotopologue abundances (M+0 ... M+n) ready for binomial model fitting
- anndata objects saved per slide by sc_analysis.py containing corrected distributions

## How to apply

For each fatty acid in each ablation mark (single-cell SpaceM data) or each pixel (mouse-brain MALDI-AIF data), apply natural isotope abundance correction to the raw isotopologue intensities extracted with pyimzML via extract_isotopologue_ion_images.py or 00_extract_ion_images.py. After correction, normalize the isotopologue distribution to fractional abundances over M+0 ... M+16 for each fatty acid in each cell. The corrected and normalized distributions are the required input format for fit_binomial in the 13C_binomial package, which expects a numpy array of normalized, natural-abundance-corrected intensities and a model type (C16 or C18; C18 accounts for an M+2 peak arising from unlabeled palmitate + two natural 13C). In the single-cell workflow this step is executed inside sciso/sc_analysis.py, per slide, before median-averaging of signal per cell per annotated m/z. Quality-control filtering (cell size, number of detected ions, total ion intensity) is applied afterwards to the aggregated anndata objects.

## Related tools

- **pyimzML** (Parses imzML files to produce the raw per-pixel/per-ablation-mark isotopologue ion images that feed the correction step) — https://github.com/alexandrovteam/pyimzML
- **13C-SpaceM** (Runs sciso/sc_analysis.py, which performs natural isotope abundance correction for each fatty acid in each ablation mark before median aggregation and normalization) — https://github.com/Buglakova/13C-SpaceM
- **13C_binomial** (Consumes the corrected and normalized isotopologue intensities via fit_binomial (C16/C18 models); the correction is a prerequisite for its binomial fit) — https://github.com/Buglakova/13C_binomial
- **Scanpy** (Stores corrected single-cell isotopologue distributions in pickled anndata objects per slide)

## Examples

```
python sciso/sc_analysis.py $data_dir/20220411_AB_DKFZACLYac/slide1/spacem_data $data_dir/20220411_AB_DKFZACLYac/slide1/anndata
```

## Evaluation signals

- Corrected palmitate isotopologue distributions in control cells yield pseudo-bulk M+0 fraction ~0.6, comparable to bulk LC-MS ~0.3 — deviation indicates mis-correction
- After correction and normalization, isotopologue fractions sum to 1 across M+0 ... M+16 per fatty acid per cell/pixel
- Corrected distributions from ACLYkd wells show the expected shift toward lower-mass isotopologues (higher M+0, lower fractional acetyl-CoA labelling) versus control wells
- fit_binomial success flag: fit_success=1 and the reported p values for glioma tumour pixels fall in the 0.45-0.70 range, confirming correction quality upstream
- M+2 enrichment in stearate/oleate elongation readouts and absence of 20:5 labelling remain detectable after correction, matching tumour-vs-brain validation patterns

## Limitations

- Correction cannot compensate for MALDI-AIF lipid-class selectivity: only fatty acids from a subset of lipid classes effectively ionized in negative mode (primarily acidic glycerophospholipids) are analyzed
- Does not address isotopic non-steady-state: different fatty acid pools may not reach isotopic steady state within the 48-h 13C6-glucose labelling period, so corrected labelling fractions must be interpreted alongside pool size data
- If overall substrate usage is very low (hypoxia/normoxia case), corrected distributions still cannot support a reliable binomial fit or uptake-fraction estimate
- Method yields relative, not absolute, substrate contribution to the lipogenic acetyl-CoA pool since pool size cannot be quantified
