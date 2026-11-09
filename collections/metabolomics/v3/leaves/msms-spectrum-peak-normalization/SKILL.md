---
name: msms-spectrum-peak-normalization
description: Use when an experimental MS/MS spectrum (peak m/z and intensity pairs)
  must be scored against theoretical fragment peak lists from candidate molecules,
  and intensity-scale differences between spectra would bias the match score.
when_to_use_negative:
- When only precursor/MS1 peak m/z values are needed (candidate retrieval there uses
  the --delta_mz ppm tolerance, not MS/MS intensity normalization)
- When the spectrum has already been normalized or the downstream scorer expects raw
  intensities (re-normalizing would double-scale the intensity weights)
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ChemDistiller
  role: Engine whose msspectra.spectrum module defines the peak/spectrum handling
    and whose FragScorer consumes the normalized spectrum for intensity-weighted matching
    against pre-calculated fragmentation patterns
  repo: https://bitbucket.org/iAnalytica/chemdistillerpython
- name: NumPy
  role: Array library used to rescale the peak intensity vector so intensities sum
    to 1.0
- name: SciPy
  role: Required dependency of the ChemDistiller annotation engine within which the
    normalization is embedded
- name: RDKit
  role: Optional; used to parse candidate structures into molecular graphs for the
    fragment enumeration whose peaks are matched against the normalized spectrum
  repo: http://www.rdkit.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/bty080
    title: 'ChemDistiller: an engine for metabolite annotation in mass spectrometry'
  source_content_hash: sha256:839ce8c65bda697bbb4b14b6db94b53be4a37b1686a105876068330ad61c1034
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemDistiller
  - NumPy
  - RDKit
  - SciPy
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# msms-spectrum-peak-normalization

## Summary

Normalizes an experimental tandem MS spectrum's peak intensities so they sum to 1.0, mirroring the ChemDistiller msspectra spectrum handling, so that candidate-scoring against pre-calculated fragmentation patterns is intensity-weighted and comparable across spectra.

## When to use

Use when an experimental MS/MS spectrum (peak m/z and intensity pairs) must be scored against theoretical fragment peak lists from candidate molecules, and intensity-scale differences between spectra would bias the match score.

## When NOT to use

- When only precursor/MS1 peak m/z values are needed (candidate retrieval there uses the --delta_mz ppm tolerance, not MS/MS intensity normalization)
- When the spectrum has already been normalized or the downstream scorer expects raw intensities (re-normalizing would double-scale the intensity weights)

## Inputs

- Experimental tandem MS spectrum as peak m/z and intensity pairs (ChemDistiller internal hierarchical format, MassBankTestSpectra test files)
- Theoretical fragment peak list per candidate compound (exact masses of recursively enumerated fragments, e.g. from the FragScorer bond-breaking enumerator)

## Outputs

- Intensity-normalized MS/MS spectrum (intensities summing to 1.0)
- Per-candidate intensity-weighted fragment-match score, exposed for aggregation by a totalscore-style combiner

## How to apply

Read the experimental tandem MS spectrum as peak m/z and intensity pairs, handling peaks and spectra the way chemdistiller.msspectra does (msspectra.spectrum module). Rescale the intensity vector so the intensities sum to 1.0 before any matching — in FragScorer this normalized spectrum is matched against the pre-calculated fragmentation pattern of each candidate compound retrieved from the compound database (tens of millions of entries with pre-calculated 'fingerprints' and fragmentation). The rationale is that FragScorer performs an intensity-weighted match between theoretical fragment exact masses and experimental peaks, so raw absolute intensities must not be compared directly; only relative, normalized weights are meaningful. Keep the normalization inside a single importable scoring module whose score is exposed for aggregation by a totalscore-style combiner, and validate it with a smoke test on one molecule and one spectrum (e.g. from the MassBank test spectra shipped in ChemDistiller).

## Related tools

- **ChemDistiller** (Engine whose msspectra.spectrum module defines the peak/spectrum handling and whose FragScorer consumes the normalized spectrum for intensity-weighted matching against pre-calculated fragmentation patterns) — https://bitbucket.org/iAnalytica/chemdistillerpython
- **NumPy** (Array library used to rescale the peak intensity vector so intensities sum to 1.0)
- **SciPy** (Required dependency of the ChemDistiller annotation engine within which the normalization is embedded)
- **RDKit** (Optional; used to parse candidate structures into molecular graphs for the fragment enumeration whose peaks are matched against the normalized spectrum) — http://www.rdkit.org/

## Examples

```
import numpy as np; spectrum_intensities = spectrum_intensities / np.sum(spectrum_intensities)  # FragScorer: intensities sum to 1.0
```

## Evaluation signals

- After normalization, sum of the spectral intensities equals 1.0 within floating-point tolerance
- The m/z values are unchanged by normalization; only intensities are rescaled
- Smoke test on one molecule and one spectrum reproduces a finite, non-degenerate FragScorer score that feeds into the totalscore-style combiner
- Running the packaged engine in test mode (python annotate.py --test) over the 863 MassBank test spectra completes and prints retrieval statistics

## Limitations

- ChemDistiller v0.1 spectra must be provided in its own internal hierarchical format; converters from other formats were not yet implemented
- RDKit is optional and only used to generate 2D structure images for the report, so structure parsing must be guarded where RDKit is unavailable
- Normalization alone does not establish candidate correctness — retrieval statistics require compounds with known structures, as in the MassBank test run

## Evidence

- [abstract] normalizes the experimental MS/MS spectrum so that its intensities sum to 1.0: "combines automated large-scale annotation of metabolites using tandem MS data with a compiled database containing tens of millions of compounds with pre-calculated 'fingerprints' and fragmentation"
- [methods] spectrum handling module: ".. automodule:: chemdistiller.msspectra.spectrum"
- [readme] dependencies and test data: "__ChemDistiller__ supports Python 2 and 3 (64-bit version recommended) and requires _SciPy_, _NumPy_, _h5py_ libraries."
- [readme] validation via test run: [evidence span withheld — anchor status: external_doc]
