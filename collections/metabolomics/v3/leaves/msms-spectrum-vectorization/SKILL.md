---
name: msms-spectrum-vectorization
description: Use when filtered MS/MS spectra (matchms Spectrum objects or m/z + intensity
  arrays) must be turned into fixed-width numerical vectors — e.g., to feed a Siamese
  neural network for embedding creation or pairwise similarity scoring.
when_to_use_negative:
- Spectra have not yet been filtered (peak removal, 1000-peak cap, square-root intensity
  transform must happen first per the paper's preprocessing)
- The analysis needs the original discrete peak list rather than a quantized vector
  — e.g., classical peak-matching measures like modified Cosine that align individual
  peaks
- Peaks lie outside the 10–1000 m/z binning window and must be retained, since binning
  is restricted to that range
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: matchms
  role: Python library for mass spectra import, handling, filtering and comparison
    used to load and prepare spectra before binning
  repo: https://github.com/matchms/ms2deepscore
- name: MS2DeepScore
  role: Siamese neural network whose 'base' network consumes the binned 10,000-element
    spectrum vectors to produce 200-dimensional embeddings
  repo: https://github.com/matchms/ms2deepscore
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-021-00558-4
    title: 'MS2DeepScore: a novel deep learning similarity measure to compare tandem
      mass spectra'
  source_content_hash: sha256:f7b185c29d3fbf9848ae30f45b7af3e93e36242b23891da7c162d9026f1852f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/ms2deepscore
  tools:
  - MS2DeepScore
  - matchms
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# msms-spectrum-vectorization

## Summary

Converts MS/MS peak lists into fixed-length 10,000-element numerical vectors by binning peaks into equal-width bins spanning 10–1000 m/z, keeping the highest peak intensity per bin. This is the first stage of MS2DeepScore input preparation: binned spectra feed the 'base' Siamese network whose inference produces the 200-dimensional embeddings.

## When to use

Use when filtered MS/MS spectra (matchms Spectrum objects or m/z + intensity arrays) must be turned into fixed-width numerical vectors — e.g., to feed a Siamese neural network for embedding creation or pairwise similarity scoring.

## When NOT to use

- Spectra have not yet been filtered (peak removal, 1000-peak cap, square-root intensity transform must happen first per the paper's preprocessing)
- The analysis needs the original discrete peak list rather than a quantized vector — e.g., classical peak-matching measures like modified Cosine that align individual peaks
- Peaks lie outside the 10–1000 m/z binning window and must be retained, since binning is restricted to that range

## Inputs

- Filtered MS/MS spectra as matchms Spectrum objects or numpy arrays of peak m/z and square-root transformed intensities (peaks <0.1% max intensity removed, max 1000 peaks)
- Bin definition: 10,000 equal-width bins from 10 to 1000 m/z

## Outputs

- binned_spectra.npy — 2D numpy array of shape (n_spectra, 10000), each row the binned vector of one spectrum
- Binned vectors ready for base neural network inference to produce 200-dimensional MS2DeepScore embeddings

## How to apply

First ensure spectra are already filtered — peaks with intensities <0.1% of the maximum peak intensity removed, at most the 1000 highest-intensity peaks retained, and intensities square-root transformed. Then define 10,000 equally-sized bins ranging from 10 to 1000 m/z, following the paper's binning scheme. For each spectrum, assign each peak to its bin and set that bin's value to the highest peak intensity it contains, leaving bins without peaks empty. Assemble all binned spectra into a single 2D numpy array (spectra x 10,000 bins) and save it, e.g. as binned_spectra.npy. The rationale is that this binning plus base-network inference constitutes the embedding creation step that precedes pairwise Tanimoto score prediction, and the fixed 10,000-width representation is what the base network consumes.

## Related tools

- **matchms** (Python library for mass spectra import, handling, filtering and comparison used to load and prepare spectra before binning) — https://github.com/matchms/ms2deepscore
- **MS2DeepScore** (Siamese neural network whose 'base' network consumes the binned 10,000-element spectrum vectors to produce 200-dimensional embeddings) — https://github.com/matchms/ms2deepscore

## Examples

```
import numpy as np; edges = np.linspace(10, 1000, 10001); binned = np.zeros((len(spectra), 10000)); np.save('binned_spectra.npy', binned)
```

## Evaluation signals

- Each binned spectrum vector is exactly 10,000 elements wide
- On the training data, 9948 of the 10,000 bins are non-empty, matching the reported bin occupancy
- Bins containing no peaks are left empty (zero) rather than filled with summed or interpolated intensity
- Peak intensities were square-root transformed before binning, and the 1000-peak cap was applied
- Assembled array shape is (n_spectra, 10000) and loads cleanly from binned_spectra.npy via numpy

## Limitations

- Binning restricted to 10–1000 m/z discards peaks outside that range and quantizes peaks within a bin to a single (highest) intensity value
- The authors could not significantly decrease the Tanimoto prediction error with less restrictive binning or deeper/wider networks, so alternative binning schemes may not help
- Binning is only one stage: embeddings also require base-network inference, and scores are not deterministic and may change when a new model is trained

## Evidence

- [discussion] Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z.: "Spectrum peaks were binned in 10,000 equally-sized bins ranging from 10 to 1000 m/z."
- [discussion] removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks.: "removing peaks with intensities < 0.1% of the maximum peak intensity and limiting the maximum number of peaks to the 1000 highest intensity peaks."
- [results] The embedding creation includes the mass spectrum binning as well as the inference step with the ‘base’ neural network (Fig. 1): "The embedding creation includes the mass spectrum binning as well as the inference step with the ‘base’ neural network (Fig. 1)"
- [readme] To calculate chemical similarity scores, MS2DeepScore first calculates an embedding (vector) representing each spectrum.: [evidence span withheld — anchor status: external_doc]
