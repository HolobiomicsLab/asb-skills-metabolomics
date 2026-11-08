---
name: mz-range-bin-interval-construction
description: 'Use when MS/MS spectra (already filtered: <0.1% max-intensity peaks
  removed, ≤1000 peaks, square-root transformed intensities) must be converted into
  fixed-width numerical vectors as input to the MS2DeepScore base network.'
when_to_use_negative:
- Spectra with peaks outside the 10–1000 m/z range, which fall outside the binning
  grid and would be silently dropped
- Raw, unfiltered spectra — the <0.1% intensity cutoff, top-1000-peak limit, and square-root
  intensity transform must be applied first
- Analyses using an alternative unsupervised spectral similarity metric such as Spec2Vec,
  which does not rely on this fixed 10,000-bin representation
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MS2DeepScore
  role: Consumes the 10,000-bin binned spectrum vectors as input to the Siamese 'base'
    neural network; binning plus inference constitutes embedding creation
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Python library used for MS/MS spectrum import, handling, and filtering prior
    to binning
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:c734ef8d13453793f768b9d02233739b11b118ecfc894cdfd61309d270e9374b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# m/z-range bin interval construction

## Summary

Constructs 10,000 equal-width bin intervals spanning 10–1000 m/z and converts each filtered MS/MS spectrum into a fixed-length vector by retaining the highest peak intensity per bin, the input format required by MS2DeepScore's Siamese base network before embedding creation.

## When to use

Use when MS/MS spectra (already filtered: <0.1% max-intensity peaks removed, ≤1000 peaks, square-root transformed intensities) must be converted into fixed-width numerical vectors as input to the MS2DeepScore base network.

## When NOT to use

- Spectra with peaks outside the 10–1000 m/z range, which fall outside the binning grid and would be silently dropped
- Raw, unfiltered spectra — the <0.1% intensity cutoff, top-1000-peak limit, and square-root intensity transform must be applied first
- Analyses using an alternative unsupervised spectral similarity metric such as Spec2Vec, which does not rely on this fixed 10,000-bin representation

## Inputs

- Filtered MS/MS spectra (peaks <0.1% of max intensity removed, max 1000 highest-intensity peaks retained, square-root transformed intensities), loaded as numpy arrays
- Bin interval definition: 10,000 equally-sized bins spanning 10 to 1000 m/z

## Outputs

- binned_spectra.npy: a 2D numpy array of shape (n_spectra, 10,000) where each row is a spectrum's binned peak-intensity vector
- Binned spectra feeding the MS2DeepScore 'base' neural network to produce 200-dimensional embeddings

## How to apply

Define 10,000 equally-sized bin intervals ranging from 10 to 1000 m/z, following the paper's binning scheme; the interval width is (1000-10)/10000 = 0.099 m/z per bin. For each spectrum, assign every peak's m/z to its corresponding bin and set that bin's value to the highest peak intensity it contains (peaks <0.1% of maximum intensity and beyond the top-1000 by intensity have already been removed and intensities square-root transformed). Bins without peaks are left empty (zero), producing a 10,000-element vector per spectrum. Assemble all binned spectra into a 2D numpy array (spectra × 10,000 bins) and save as binned_spectra.npy; this binning plus inference with the 'base' neural network constitutes the embedding creation step, which is computationally far more expensive than the subsequent cosine-similarity score computation. Rationale: the fixed-width binned representation lets the Siamese network consume peak m/z positions and intensities only, without requiring molecular mass or formula as in DeepMass.

## Related tools

- **MS2DeepScore** (Consumes the 10,000-bin binned spectrum vectors as input to the Siamese 'base' neural network; binning plus inference constitutes embedding creation) — https://github.com/matchms/ms2deepscore
- **matchms** (Python library used for MS/MS spectrum import, handling, and filtering prior to binning) — https://github.com/matchms/ms2deepscore

## Examples

```
import numpy as np; edges = np.linspace(10, 1000, 10001); binned = np.zeros((len(spectra), 10000)); [np.maximum.at(binned[i], np.digitize(p.mz, edges)-1, p.intensities) for i, s in enumerate(spectra) for p in [s.peaks]]; np.save('binned_spectra.npy', binned)
```

## Evaluation signals

- Each binned vector is exactly 10,000 elements wide
- On the paper's training data, 9948 of the 10,000 bins are non-empty, matching the reported bin occupancy
- All bins are equally-sized and collectively span exactly 10 to 1000 m/z (0.099 m/z bin width)
- Bins without peaks remain empty (zero) rather than being interpolated or summed
- The output 2D numpy array has shape (n_spectra, 10,000) and loads cleanly from binned_spectra.npy

## Limitations

- Only the highest peak intensity per bin is retained, so multiple fragments within one 0.099 m/z bin lose distinguishing information
- The authors could not significantly decrease the overall Tanimoto prediction error with less restrictive binning, suggesting finer bin intervals do not improve the model
- Binned spectra outside the 10–1000 m/z window cannot be represented; the model was trained on GNPS-derived spectra of varying instrumentation and compound-class balance

## Evidence

- [discussion] Binning scheme definition: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [results] Role of binning in embedding creation: "The embedding creation includes the mass spectrum binning as well as the inference step with the ‘base’ neural network (Fig. 1)"
- [discussion] Spectrum preprocessing preceding binning: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [readme] Embeddings as input representation purpose: [evidence span withheld — anchor status: external_doc]
