---
name: ann-fingerprint-prediction
description: Use when an unknown LC-MS/MS analyte spectrum (precursor mass + m/z/intensity
  peak list) must be annotated by predicting a 528-bit fingerprint and ranking candidate
  compounds retrieved by precursor m/z or molecular formula.
when_to_use_negative:
- The compound is a known unknown with a curated reference MS/MS spectrum available
  — direct spectral library matching against libraries such as MoNA, HMDB, METLIN,
  or GNPS is the appropriate approach instead.
- The candidate list is empty or cannot be retrieved (no precursor m/z hit within
  10 PPM and no molecular formula query to PubChem) — there is nothing to rank.
- The spectrum consists of fewer than five peaks with relative intensity above 2%
  — such spectra are removed by the scaling and filtering step.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetFID
  role: 'Core skill implementation: CNN that predicts the full 528-bit fingerprint
    from the binned MS/MS spectrum and ranks candidates'
  repo: https://github.com/aohongyu/MetFID
- name: TensorFlow
  role: Keras/TensorFlow backend used to load and run the pretrained MetFID_CNN_40088_5618.h5
    model and to train ANNs (two hidden ReLU layers, sigmoid output, Adam, 30 epochs,
    batch size 100)
- name: OpenBabel
  role: Computes MACCS, FP3, and FP4 fingerprints for candidate compounds, assembled
    into the 528 binary-entry target vector
- name: PyFingerprint
  role: Python wrapper used to obtain OpenBabel fingerprints (fp3, fp4) for candidate
    structures from InChIKeys
  repo: https://github.com/hcji/PyFingerprint
- name: PubChemPy
  role: Queries the PubChem database by molecular formula to retrieve candidate compound
    lists when m/z search is not used
- name: MetaboQuest
  role: Retrieves candidate compounds by precursor m/z search with 10 PPM tolerance
    and H+ adduct assumption
  repo: https://omicscraft.com/MetaboQuest
- name: tabulate
  role: Formats the per-block ranked candidate tables written to testing_compound_prediction.txt
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_007
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:268e619a3ad1c6bc202637346a831f8ace8062c42f2c0823606179728fa4046c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - MetFID
  - MetaboQuest
  - MoNA
  - NIST 17
  - OpenBabel
  - PubChemPy
  - PyFingerprint
  - RDKit
  - TensorFlow
  - tabulate
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# ann-fingerprint-prediction

## Summary

Predicts a full 528-bit compound fingerprint (MACCS + FP3 + FP4) from an MS/MS spectrum with a single ANN/CNN model, then ranks candidate compounds by fingerprint similarity for metabolite annotation. This avoids the multi-model per-block design of earlier fingerprint predictors and neglect of latent relationships between fingerprint digits.

## When to use

Use when an unknown LC-MS/MS analyte spectrum (precursor mass + m/z/intensity peak list) must be annotated by predicting a 528-bit fingerprint and ranking candidate compounds retrieved by precursor m/z or molecular formula.

## When NOT to use

- The compound is a known unknown with a curated reference MS/MS spectrum available — direct spectral library matching against libraries such as MoNA, HMDB, METLIN, or GNPS is the appropriate approach instead.
- The candidate list is empty or cannot be retrieved (no precursor m/z hit within 10 PPM and no molecular formula query to PubChem) — there is nothing to rank.
- The spectrum consists of fewer than five peaks with relative intensity above 2% — such spectra are removed by the scaling and filtering step.

## Inputs

- MS/MS peak lists in testing_compound.txt format: precursor mass and ionization mode header line followed by m/z–intensity pairs per block (#digit separators)
- Candidate compound lists as InChIKeys in inchikey_list.txt, one list per test spectrum, block counters matching testing_compound.txt
- Pretrained CNN model file MetFID_CNN_40088_5618.h5 placed in the MetFID root directory

## Outputs

- testing_compound_prediction.txt in the _files folder: per-block tables of candidate InChIKeys with similarity scores ranked in descending order
- Predicted 528-bit fingerprint vector per input spectrum (MACCS + FP3 + FP4)
- Top-k annotation accuracy metrics on structure-disjoint testing compounds

## How to apply

Preprocess the spectrum exactly as in training: scale peak intensities to 0-100, denoise by removing peaks with intensity below 10, compute neutral-loss features (precursor m/z minus each peak's m/z), and bin the m/z range into prespecified integer bins; for training corpora, additionally merge spectra of the same compound (via InChIKey) across collision energies into single peaks when m/z differences are ≤0.1. Load the pretrained Keras/TensorFlow model MetFID_CNN_40088_5618.h5 and predict the 528-bit fingerprint in a single pass (sigmoid output trained with binary cross-entropy). Compute the 528-digit fingerprint for each candidate compound with OpenBabel fingerprints (via PyFingerprint), then score each candidate with δ(f′,f) = 1 − Σ|f′−f|/n and rank in descending order; the top-ranked candidate is the annotation. Retrieve candidates beforehand via MetaboQuest precursor m/z search (10 PPM, H+ adduct) or via PubChemPy formula queries against PubChem. Evaluate by checking whether the true compound appears in the top-k ranked list (top-k annotation accuracy).

## Related tools

- **MetFID** (Core skill implementation: CNN that predicts the full 528-bit fingerprint from the binned MS/MS spectrum and ranks candidates) — https://github.com/aohongyu/MetFID
- **TensorFlow** (Keras/TensorFlow backend used to load and run the pretrained MetFID_CNN_40088_5618.h5 model and to train ANNs (two hidden ReLU layers, sigmoid output, Adam, 30 epochs, batch size 100))
- **OpenBabel** (Computes MACCS, FP3, and FP4 fingerprints for candidate compounds, assembled into the 528 binary-entry target vector)
- **PyFingerprint** (Python wrapper used to obtain OpenBabel fingerprints (fp3, fp4) for candidate structures from InChIKeys) — https://github.com/hcji/PyFingerprint
- **PubChemPy** (Queries the PubChem database by molecular formula to retrieve candidate compound lists when m/z search is not used)
- **MetaboQuest** (Retrieves candidate compounds by precursor m/z search with 10 PPM tolerance and H+ adduct assumption) — https://omicscraft.com/MetaboQuest
- **tabulate** (Formats the per-block ranked candidate tables written to testing_compound_prediction.txt)

## Examples

```
python3 main.py  # in MetFID root, with MetFID_CNN_40088_5618.h5 and _files/testing_compound.txt + _files/inchikey_list.txt present
```

## Evaluation signals

- Output file testing_compound_prediction.txt exists in _files and contains per-block tables of InChIKeys with similarity scores sorted in descending order.
- Block counters (#1, #2, …) match between testing_compound.txt and inchikey_list.txt; mismatch raises an error per the README.
- On structure-disjoint NIST testing spectra with MetaboQuest-retrieved candidates, recomputed top-1 annotation accuracy is ~50%, matching the reported rate.
- Recomputed annotation accuracy exceeds MetFrag and ChemDistiller baselines by more than 5% on the structure-disjoint NIST testing set.
- Predicted target vectors are exactly 528 binary entries (MACCS + FP3 + FP4) and preprocessing applied intensity scaling to 0-100 with denoising cutoff 10.

## Limitations

- Structurally similar candidates cannot be distinguished: in the in-house HCC case (m/z = 758.5697, 12 candidates), four candidates tied at the highest score, so additional ranking criteria are needed.
- Collision energy strongly affects fragmentation patterns; 30 eV was a reasonable cutoff, and separate ANNs for low (≤30 eV) and high (≥31 eV) energies outperformed the combined model on low-energy testing spectra.
- Testing spectra are processed per collision energy without merging across energies, so the merged-spectra training regime must not be applied naively at prediction time.
- The pretrained MetFID_CNN_40088_5618.h5 must be downloaded separately; the pipeline requires Python 3.5+, PyFingerprint, PubChemPy 1.0.4, Open Babel 3.1.1, and tabulate 0.8.9.

## Evidence

- [intro] fingerprint prediction from spectrum: "A trained machine learning model receives an MS/MS spectrum of an unknown compound as input and predicts its fingerprint as output."
- [methods] 528-bit fingerprint definition: "Specifically, MACCS, FP3, and FP4 fingerprints were determined and assembled into a vector comprising of 528 binary entries"
- [methods] scaling and filtering of spectra: "We scaled the peak intensities such that all intensity values are between 0 and 100. Spectra that consisted of fewer than five peaks with relative intensity above 2% were removed"
- [methods] denoising rule: "we removed peaks whose intensity values were smaller than that of the highest intensity, subsequently removing all peaks with intensity lower than 10"
- [methods] training-phase rationale: [evidence span withheld — anchor status: not_found]
- [readme] pretrained model requirement: "please download the latest CNN model -- [MetFID_CNN_40088_5618.h5][5] and put it in the root directory of MetFID."
- [readme] output artifact: "There will be an output file `testing_compound_prediction.txt` be created in the folder `_files`."
