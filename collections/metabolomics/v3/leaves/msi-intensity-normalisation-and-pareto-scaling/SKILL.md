---
name: msi-intensity-normalisation-and-pareto-scaling
description: Use when an imzML-derived, annotated MSI feature table (e.g. image.norm.csv
  from massPix) must be prepared as input for PCA or clustering, and pixel-to-pixel
  intensity variation needs correction before scaling.
when_to_use_negative:
- The input is already a normalised and scaled feature table (e.g. a previous image.norm.csv
  run with identical settings) — re-running normalise/centreScale would distort intensities.
- Single ion imaging only (imageSlice), where absolute per-ion intensities, not cross-pixel
  multivariate comparability, are the target.
- Analyses requiring absolute quantification across tissue slices — TIC or median
  normalisation only corrects relative intensity variation, not absolute amount.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: massPix
  role: Provides the normalise function (median / TIC / standard-ion normalisation)
    and centreScale function (centering and Pareto scaling, scale.type='cs') applied
    before PCA or k-means clustering.
  repo: https://github.com/hallz/massPix
- name: R
  role: Runtime environment in which massPix's normalisation and scaling functions
    are executed within the massPix processing_script.R wrapper.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-017-1252-5
    title: 'massPix: an R package for annotation and interpretation of mass spectrometry
      imaging data for lipidomics'
  source_content_hash: sha256:4c5f18cfe6696a5f61781e383807d136f1008e8a129eefe5b8be586d083eba6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hallz/massPix
  tools:
  - R
  - massPix
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# msi-intensity-normalisation-and-pareto-scaling

## Summary

Normalises MSI ion intensities (median, total ion count, or standard ions) and then centre/Pareto-scales the pixel-by-feature matrix in massPix so that downstream PCA or k-means clustering of tissue regions is on a comparable intensity basis.

## When to use

Use when an imzML-derived, annotated MSI feature table (e.g. image.norm.csv from massPix) must be prepared as input for PCA or clustering, and pixel-to-pixel intensity variation needs correction before scaling.

## When NOT to use

- The input is already a normalised and scaled feature table (e.g. a previous image.norm.csv run with identical settings) — re-running normalise/centreScale would distort intensities.
- Single ion imaging only (imageSlice), where absolute per-ion intensities, not cross-pixel multivariate comparability, are the target.
- Analyses requiring absolute quantification across tissue slices — TIC or median normalisation only corrects relative intensity variation, not absolute amount.

## Inputs

- Normalised/annotated MSI image matrix from massPix (image.norm.csv: lipid annotations, m/z and intensities per pixel)
- imzML/.ibd data file pair parsed via imzMLConverter.jar (e.g. test_POS.imzML and test_POS.ibd from MetaboLights MTBLS487)
- Normalisation choice: norm.type = 'TIC' | 'median' | standard ions; scaling choice: scale.type = 'cs' (centre + Pareto)

## Outputs

- image.norm.csv — full normalised image data matrix (lipid annotations, m/z and intensities)
- image.norm_short.csv — lipid annotations and m/z only
- Centre- and Pareto-scaled pixel-by-feature matrix ready for imagePca or cluster
- PC scores images and loadings_PC1.csv … loadings_PC5.csv when PCA follows

## How to apply

After massPix pre-processing (mzextractor, peakpicker.bin with bin.ppm=10 and thres.filter=11, deisotope, annotate, constructImage), call the normalise function on the full image matrix choosing norm.type='TIC' (total ion count), 'median', or normalisation to a set of standard ions. Then apply centreScale with scale.type='cs' to center and Pareto-scale the normalised image matrix; leave the log transform off (transform=F) unless justified. Feed the scaled matrix into imagePca (e.g. PCnum=5) or the cluster function for multivariate statistics. The rationale is that raw ion intensities in MSI spectra vary with total signal per pixel, so normalisation to TIC removes this dominant technical variation before Pareto scaling balances each lipid feature's contribution to PCA.

## Related tools

- **massPix** (Provides the normalise function (median / TIC / standard-ion normalisation) and centreScale function (centering and Pareto scaling, scale.type='cs') applied before PCA or k-means clustering.) — https://github.com/hallz/massPix
- **R** (Runtime environment in which massPix's normalisation and scaling functions are executed within the massPix processing_script.R wrapper.)

## Examples

```
results <- massPix(process=T, pca=T, thres.int=100000, thres.low=200, thres.high=1000, bin.ppm=10, thres.filter=11, ppm=3, no_isotopes=2, prop.1=0.9, prop.2=0.5, norm.type="TIC", scale.type="cs", transform=F, PCnum=5)
```

## Evaluation signals

- Output file image.norm.csv exists and contains lipid annotations, m/z values and per-pixel intensities as columns.
- Each normalised pixel's intensities are on a comparable scale — with norm.type='TIC', per-pixel ion intensity sums should be approximately constant across pixels.
- centreScale returns a scaled matrix with column (feature) means of zero and Pareto-scaled standard deviations; PCA on the unscaled matrix should give visibly different PC scores images.
- Downstream PCA (e.g. PCnum=5, princomp cor=FALSE) produces PC scores images that spatially separate the white matter, granular and molecular layers of the cerebellum as in Fig. 2c.
- Cluster/PCA functions run without errors on the scaled matrix, confirming transform=F and scale.type='cs' were accepted parameters.

## Limitations

- Normalisation choices are limited to median, total ion count, or average intensity of a set of standard ions; no other strategies are implemented.
- Pareto scaling follows centering in centreScale; applying a log transform is off by default and changing defaults for normalisation, scaling and filtering requires customising the source code.
- massPix processes one imzML file at a time, so normalisation/scaling must be repeated per file; accuracy of lipid annotations feeding into the scaled matrix depends on accurate-mass matching and should be confirmed by MS/MS.

## Evidence

- [methods] Ion intensities are then normalised either to the median or total ion count, or to the average intensity of a set of standard ions (function: normalise): "Ion intensities are then normalised either to the median or total ion count, or to the average intensity of a set of standard ions (function: normalise)"
- [readme] normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling (function: centreScale and imagePca): "or normalised intensities used to create multivariate statistical images based on k-means clustering (function: cluster) or PCA following centering and Pareto scaling (function: centreScale and"
- [readme] The final output is image.norm.csv (full normalised image data matrix - lipid annotations, m/z and intensities) and image.norm_short.csv: "The final output is image.norm.csv (full normalised image data matrix - lipid annotations, m/z and intensities) and image.norm_short.csv (containing lipid annotations and m/z only)."
- [readme] The analysis can be readily customised by replacing default parameters for filtering, normalisation and scaling, library composition, lipid assignment and image reporting.: "The analysis can be readily customised by replacing default parameters for filtering, normalisation and scaling, library composition, lipid assignment and image reporting."
