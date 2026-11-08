---
name: mass-spectra-preprocessing-and-binning
description: Use when raw MS/MS spectra (mgf, msp, mzML, mzXML, json, usi) must be
  converted into normalized, binned vectors before training MS2DeepScore or computing
  spectral embeddings; triggered by GNPS-style peak lists with heterogeneous intensity
  scales.
when_to_use_negative:
- Input spectra are already binned/vectorized MS2DeepScore embeddings or model outputs
  — re-filtering would alter the input distribution the network was trained on
- Analysis requires untransformed raw intensities or m/z values finer than the 10,000-bin
  resolution (e.g. manual peak annotation at exact m/z)
- Working with fingerprints/structures only (e.g. computing Tanimoto scores from SMILES
  with RDKit) — no spectra are involved
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Provides the recommended spectrum cleaning pipeline applied before binning/training
  repo: https://github.com/matchms/matchms
- name: MS2DeepScore
  role: 'Consumer of the binned spectra: its ''base'' neural network ingests the binned
    vectors to create 200-dimensional embeddings for Tanimoto-score prediction'
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:f90f64c2098ffd18f6746d86e5e1062d7bbccc85cc5620516869894aabe9af9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - MS2DeepScore
  - RDKit
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mass-spectra-preprocessing-and-binning

## Summary

Preprocess GNPS-derived MS/MS spectra (peak-intensity filtering, peak-count capping, square-root intensity transformation) and bin peaks into 10,000 equal-width m/z bins (10–1000) to produce fixed-length vectors suitable for MS2DeepScore Siamese-network training and Tanimoto-score prediction.

## When to use

Use when raw MS/MS spectra (mgf, msp, mzML, mzXML, json, usi) must be converted into normalized, binned vectors before training MS2DeepScore or computing spectral embeddings; triggered by GNPS-style peak lists with heterogeneous intensity scales.

## When NOT to use

- Input spectra are already binned/vectorized MS2DeepScore embeddings or model outputs — re-filtering would alter the input distribution the network was trained on
- Analysis requires untransformed raw intensities or m/z values finer than the 10,000-bin resolution (e.g. manual peak annotation at exact m/z)
- Working with fingerprints/structures only (e.g. computing Tanimoto scores from SMILES with RDKit) — no spectra are involved

## Inputs

- Raw MS/MS peak lists (mgf, msp, mzML, mzXML, json, usi) e.g. from GNPS
- Zenodo record 4699300 cleaned GNPS dataset (210,407 spectra, 184,698 annotated with InChIKey and SMILES and/or InChI)

## Outputs

- Per-spectrum binned intensity vectors of length 10,000 covering 10–1000 m/z with square-root transformed intensities
- Cleaned spectra with at most 1000 peaks each, low-intensity peaks (<0.1% of maximum) removed

## How to apply

Clean each spectrum by removing peaks with intensities below 0.1% of the maximum peak intensity and capping the spectrum at the 1000 highest-intensity peaks, then square-root transform the remaining intensities. Bin the peaks into 10,000 equally-sized bins spanning 10 to 1000 m/z, yielding a fixed-length vector per spectrum regardless of instrument of origin. When working with the reference pipeline, start from the cleaned 210,407-spectrum GNPS dataset on Zenodo record 4699300 and use the matchms cleaning pipeline recommended by the ms2deepscore repository. The resulting binned spectra feed the Siamese network's 'base' neural network together with Tanimoto labels computed from RDKit Daylight 2048-bit fingerprints; this normalization is what lets a single model trained on ~102,536 spectra of 15,062 InChIKeys generalize to unseen compounds.

## Related tools

- **matchms** (Provides the recommended spectrum cleaning pipeline applied before binning/training) — https://github.com/matchms/matchms
- **MS2DeepScore** (Consumer of the binned spectra: its 'base' neural network ingests the binned vectors to create 200-dimensional embeddings for Tanimoto-score prediction) — https://github.com/matchms/ms2deepscore

## Examples

```
from matchms.Pipeline import Pipeline, create_workflow; from matchms.filtering.default_pipelines import DEFAULT_FILTERS; pipeline = Pipeline(create_workflow(query_filters=DEFAULT_FILTERS)); pipeline.run('pesticides.mgf')
```

## Evaluation signals

- After filtering, every spectrum contains at most 1000 peaks and no peak with intensity <0.1% of its maximum peak intensity
- Binned vectors all have length 10,000 with bin edges spanning exactly 10 to 1000 m/z
- Intensity values in the binned vectors are square-root transformed (max value at most sqrt of original max intensity)
- Processing the reference Zenodo 4699300 dataset should account for its 210,407 spectra without dropping or duplicating entries
- Downstream sanity check: model trained on the preprocessed data reaches RMSE between 0.13 and 0.2 on the 3601-spectrum test set of 500 unseen compounds

## Limitations

- Binning and filtering parameters were tuned for GNPS-derived spectra of mixed instrumentation and varying quality; spectra outside the 10–1000 m/z range are not represented in the binned vectors
- The preprocessing is coupled to the trained model: scores are not deterministic and may change when a new model is trained with different parameters or training data
- Deepening/widening the network or using less restrictive binning did not significantly decrease the overall Tanimoto prediction error, so re-tuning the binning resolution is unlikely to fix poor accuracy on extreme similarity bins (highest/lowest)

## Evidence

- [discussion] Peak filtering removes <0.1% intensity peaks and caps at 1000 peaks: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [discussion] Binning specification: 10,000 equal-width bins over 10–1000 m/z: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [readme] README recommends matchms cleaning pipeline before training: [evidence span withheld — anchor status: external_doc]
- [discussion] Square-root transform and augmentation context from workflow: "we applied three forms of data augmentation on the binned spectra."
