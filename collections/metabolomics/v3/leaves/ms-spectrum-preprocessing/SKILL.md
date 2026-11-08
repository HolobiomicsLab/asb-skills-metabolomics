---
name: ms-spectrum-preprocessing
description: Use when tandem MS test spectra (e.g. NPLIB1, MassSpecGym) must be downloaded,
  processed into peak lists, and prepared as input for formula annotation tools or
  de novo structure generation.
when_to_use_negative:
- Input data are already unpacked MIST compact caches (data/compact/{msg,canopus}/{train,val,test})
  that are ready for training or inference without rebuilding formula features
- The goal is fingerprint prediction or structure generation from already-processed
  spectra rather than preparing spectra for annotation
- Spectra lack high-resolution MS1 data and isotopic traces needed by tools like SIRIUS,
  and no alternative annotator (e.g. MIST-CF, which does not consider isotope distributions)
  is available
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: MIST-CF / MIST
  role: Consumes the processed peak lists to output top-5 ranked molecular formula
    candidates per spectrum; coleygroup/MIST provides compact data archives and pretrained
    msg/canopus checkpoints
  repo: https://github.com/coleygroup/MIST
- name: DiffMS
  role: Downstream de novo structure generator that consumes the processed spectra
    and the annotated top-5 formulae; data are prepared via its data_processing scripts
  repo: https://github.com/coleygroup/DiffMS
- name: BUDDY
  role: Alternative out-of-the-box formula annotator run on the same processed test
    spectra to obtain top-5 ranked formula candidates
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:654ebdb8f2f73a03717b452676fd7e9b020962a82ae9b25b7696feb9a5169ba6
  source_papers:
  - doi: 10.48550/arxiv.2502.09571
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/coleygroup/MIST
  tools:
  - BUDDY
  - DiffMS
  - MIST-CF
  - MIST-CF / MIST
  - PyTorch
  - coleygroup/MIST
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# ms-spectrum-preprocessing

## Summary

Downloads and processes benchmark tandem MS datasets (NPLIB1/CANOPUS and MassSpecGym) into clean peak lists that downstream formula annotators (MIST-CF, BUDDY) and the DiffMS de novo structure generator can consume, since reliable spectra preparation determines annotation accuracy.

## When to use

Use when tandem MS test spectra (e.g. NPLIB1, MassSpecGym) must be downloaded, processed into peak lists, and prepared as input for formula annotation tools or de novo structure generation.

## When NOT to use

- Input data are already unpacked MIST compact caches (data/compact/{msg,canopus}/{train,val,test}) that are ready for training or inference without rebuilding formula features
- The goal is fingerprint prediction or structure generation from already-processed spectra rather than preparing spectra for annotation
- Spectra lack high-resolution MS1 data and isotopic traces needed by tools like SIRIUS, and no alternative annotator (e.g. MIST-CF, which does not consider isotope distributions) is available

## Inputs

- NPLIB1 benchmark tandem MS spectra (subset of GNPS data used to train CANOPUS)
- MassSpecGym tandem MS spectra
- Per-spectrum peak lists used for formula annotation
- Ground-truth molecular formulae of test spectra (for accuracy computation)

## Outputs

- Processed test spectra with peak lists for NPLIB1 and MassSpecGym
- Per-spectrum top-5 ranked formula candidates from MIST-CF and BUDDY
- formula_predictions.csv with per-spectrum formula predictions and top-1/top-5 accuracies

## How to apply

Run the DiffMS data_processing scripts in order: 00_download_fp2mol_data.sh, 01_download_canopus_data.sh (NPLIB1, the subset of GNPS data used to train CANOPUS), 02_download_msg_data.sh (MassSpecGym), then 03_preprocess_fp2mol.sh; these require unzip and may need manual editing on non-Linux OS. Work inside a conda env with rdkit=2024.09.4, python=3.9, PyTorch 2.3.1 and torch_geometric 2.3.1. The result is per-spectrum peak lists for the NPLIB1 and MassSpecGym test splits, which are loaded alongside ground-truth formulae to feed MIST-CF and BUDDY top-5 formula annotation (written to formula_predictions.csv) and DiffMS generation. Alternatively, use the MIST compact caches, which unpack to data/compact/{msg,canopus}/{train,val,test} and are ready for inference without rebuilding formula features. Validate that every test spectrum in both datasets receives a ranked formula list and that top-1/top-5 annotation accuracies fall in the expected range.

## Related tools

- **MIST-CF / MIST** (Consumes the processed peak lists to output top-5 ranked molecular formula candidates per spectrum; coleygroup/MIST provides compact data archives and pretrained msg/canopus checkpoints) — https://github.com/coleygroup/MIST
- **DiffMS** (Downstream de novo structure generator that consumes the processed spectra and the annotated top-5 formulae; data are prepared via its data_processing scripts) — https://github.com/coleygroup/DiffMS
- **BUDDY** (Alternative out-of-the-box formula annotator run on the same processed test spectra to obtain top-5 ranked formula candidates)

## Examples

```
bash data_processing/01_download_canopus_data.sh && bash data_processing/02_download_msg_data.sh
```

## Evaluation signals

- All test spectra in both NPLIB1 and MassSpecGym receive a ranked formula list from each annotator
- Computed top-1 and top-5 formula annotation accuracies fall in the expected range for MIST-CF and BUDDY
- MIST compact data archives unpack cleanly into data/compact/{msg,canopus}/{train,val,test}
- Each processed spectrum carries a peak list that downstream annotation and DiffMS generation accept without errors

## Limitations

- MIST-CF does not consider isotope distributions, so formula inference quality differs across tools depending on available MS1 isotopic traces
- The data_processing scripts rely on unzip; on non-Linux OS the scripts may need editing or manual execution
- graph_tool and torch_geometric currently seem to conflict on MacOS (DiGress-derived codebase)

## Evidence

- [readme] MIST compact cache layout: [evidence span withheld — anchor status: external_doc]
