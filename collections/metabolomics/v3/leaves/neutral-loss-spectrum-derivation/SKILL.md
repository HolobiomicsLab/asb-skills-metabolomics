---
name: neutral-loss-spectrum-derivation
description: 'Use when preparing paired query-reference MS/MS inputs for TransExION:
  neutral-loss representations must be built alongside original-spectrum representations
  from MGF/h5py peak lists.'
when_to_use_negative:
- When processing a single isolated spectrum with no reference partner — the neutral-loss
  MDM/aligned matrix is defined per (query, reference) pair, not per spectrum
- When the goal is classic peak matching only (cosine/modified cosine), where neutral-loss
  and mass-difference features are not part of the representation
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: TransExION
  role: Defines the neutral-loss MDM/aligned-matrix construction (300 Da cut-off,
    nominal-mass splitting, defect rounding x100, pad value, CLS entry) consumed by
    the Transformer
  repo: github.com/banhdzui/TransExION
- name: pyteomics
  role: Parses the MGF training/validation spectra, keeping rounded m/z and relative-abundance
    intensities after noise filtering
- name: h5py
  role: Stores the converted MGF spectra in h5py format for large-scale processing
    before neutral-loss derivation
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:800bdde59e262a96cf264593e63a0efe0e091d3a673b745ea99fd0185341d0c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - RDKit
  - TransExION
  - h5py
  - pyteomics
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# neutral-loss-spectrum-derivation

## Summary

Converts each MS/MS spectrum into a hypothetical neutral-loss spectrum (precursor m/z minus fragment ion m/z) and then builds the neutral-loss mass difference matrix (MDM) and aligned matrix that serve as Transformer inputs for TransExION spectral similarity prediction.

## When to use

Use when preparing paired query-reference MS/MS inputs for TransExION: neutral-loss representations must be built alongside original-spectrum representations from MGF/h5py peak lists.

## When NOT to use

- When processing a single isolated spectrum with no reference partner — the neutral-loss MDM/aligned matrix is defined per (query, reference) pair, not per spectrum
- When the goal is classic peak matching only (cosine/modified cosine), where neutral-loss and mass-difference features are not part of the representation

## Inputs

- MGF-format MS/MS spectra (GNPS_MassBank_train.mgf, GNPS_MassBank_val.mgf; Zenodo record 8175528), optionally pre-converted to h5py via ms_convert_mgf2h5py.py
- Precursor m/z per spectrum (needed to compute precursor-minus-fragment neutral losses)
- Spectrum pairs (query, reference) for which MDMs and aligned matrices are required

## Outputs

- Hypothetical neutral-loss spectrum per input spectrum (precursor m/z minus each fragment ion m/z)
- Neutral-loss-based mass difference matrix (MDM) per spectrum pair
- Neutral-loss aligned matrix per spectrum pair (nominal-mass columns 0-299, mass-defect values 0-99, pad-filled gaps, leading CLS entry), saved to disk e.g. as .npz/.npy keyed by spectrum pair

## How to apply

Parse the deposited MGF training/validation spectra (GNPS_MassBank_train.mgf, GNPS_MassBank_val.mgf from Zenodo record 8175528) with pyteomics, keeping rounded m/z and relative-abundance intensities after noise filtering: drop fragments below 0.1% intensity or with m/z <10 or >1000 Da, keeping only the highest peak within each 3-Da sliding window. For each spectrum compute the hypothetical neutral-loss spectrum as precursor m/z minus each fragment ion m/z. For each (query, reference) pair compute the neutral-loss MDM of absolute mass differences, restrict to values below the 300 Da cut-off, and split each difference into its nominal mass (integer part, columns 0-299) and mass defect (fractional part rounded to two decimals then multiplied by 100, values 0-99). Fill each aligned-matrix row by positioning query fragments in nominal-mass columns, insert a pad value wherever a nominal mass is absent, and prepend a special CLS entry so the Transformer's self-attention can accumulate similarity information; the same procedure is applied to the original-spectrum MDMs in parallel.

## Related tools

- **TransExION** (Defines the neutral-loss MDM/aligned-matrix construction (300 Da cut-off, nominal-mass splitting, defect rounding x100, pad value, CLS entry) consumed by the Transformer) — github.com/banhdzui/TransExION
- **pyteomics** (Parses the MGF training/validation spectra, keeping rounded m/z and relative-abundance intensities after noise filtering)
- **h5py** (Stores the converted MGF spectra in h5py format for large-scale processing before neutral-loss derivation)

## Examples

```
python ms_convert_mgf2h5py.py --mgf GNPS_MassBank_train.mgf --out GNPS_MassBank_train.db
```

## Evaluation signals

- Neutral-loss values must equal precursor m/z minus fragment ion m/z for every fragment
- All MDM mass differences are restricted to values below the 300 Da cut-off
- Aligned-matrix cells contain mass defects rounded to two decimals and multiplied by 100, i.e. integers in the 0-99 range; absent nominal-mass columns carry the pad value; a special CLS entry precedes each aligned matrix
- Re-running the derivation on the same deposited training files yields element-wise identical MDMs and aligned matrices (agreement within the two-decimal rounding tolerance); any nonzero element-wise difference is flagged
- The neutral-loss pipeline produces the same aligned-matrix conventions as the original-spectrum pipeline applied to the same pairs

## Limitations

- Fragments below 0.1% relative intensity, with m/z <10, or above 1000 Da are discarded, and only the highest peak within a 3-Da sliding window is kept — spectra violating these assumptions cannot be reproduced faithfully
- Nominal masses absent from the MDM are represented only by a pad value, so information about missing neutral losses is not encoded beyond padding
- The mass-defect encoding quantizes to two decimals (0-99), so sub-0.01-Da defect differences are lost

## Evidence

- [intro] article-intro-mass-difference: "in addition to aligned product ions and neutral losses, the mass difference between query and reference fragments are also considered for spectral similarity"
- [abstract] article-abstract-detect-fragments: "TransExION detects related fragments between MS/MS spectra through their mass difference"
- [readme] readme-h5py-conversion: "the input data files are converted into h5py format. Our library supports a function to convert a MGF format file into h5py format."
