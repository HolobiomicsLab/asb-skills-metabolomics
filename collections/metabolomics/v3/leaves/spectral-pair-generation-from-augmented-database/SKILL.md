---
name: spectral-pair-generation-from-augmented-database
description: Use when you need to probe a trained spectral classifier's tolerance
  to chemical shift variation by synthetically shifting mixture-component NMR spectral
  pairs across a controlled ppm interval and collecting per-shift predictions.
when_to_use_negative:
- Input spectra are already experimental mixtures with unknown components — use the
  flavor mixtures benchmarking workflow instead of synthetic pair shifting
- The goal is training the pSCNN model from scratch on the augmented dataset rather
  than evaluating shift robustness of a pretrained model
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: pSCNN
  role: Pretrained pseudo Siamese CNN reloaded to predict probabilities on the 980
    varied spectral pairs
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: Python 3.8.13
  role: Runtime environment for executing the shift-variation and prediction scripts
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: TensorFlow 2.5.0-GPU
  role: Deep learning framework used to load and run the pretrained pSCNN model
- name: Anaconda
  role: Environment manager used to create the pSCNN Python 3.8.13 environment from
    requirements.txt
  repo: https://www.anaconda.com/
- name: tqdm
  role: Progress-bar dependency installed via pip for the prediction loop over varied
    spectral pairs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:03d4a8e6f26c2f4a56459cab0ebf40f2597da2c8294fc9ea01acd2d9d69a7ef3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yuxuanliao/pSCNN.git
  tools:
  - Anaconda
  - Python
  - Python 3.8.13
  - TensorFlow 2.5.0-GPU
  - pSCNN
  - tqdm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# spectral-pair-generation-from-augmented-database

## Summary

Generates 980 chemically shifted NMR spectral pairs by applying 35 chemical shift variations within +/-0.052 ppm (Table S6 scheme) to each of the 8 augmented mixture-component pairs, enabling systematic stress-testing of pSCNN robustness to chemical shift variation.

## When to use

Use when you need to probe a trained spectral classifier's tolerance to chemical shift variation by synthetically shifting mixture-component NMR spectral pairs across a controlled ppm interval and collecting per-shift predictions.

## When NOT to use

- Input spectra are already experimental mixtures with unknown components — use the flavor mixtures benchmarking workflow instead of synthetic pair shifting
- The goal is training the pSCNN model from scratch on the augmented dataset rather than evaluating shift robustness of a pretrained model

## Inputs

- Pretrained pSCNN model from Zenodo (DOI 10.5281/zenodo.6504814)
- 8 augmented mixture-component NMR spectral pairs
- Chemical shift variation scheme from Table S6 (35 variations within +/-0.052 ppm)

## Outputs

- 980 chemically varied NMR spectral pairs
- Per-variation predicted probabilities (pSCNN_shift_predictions.csv)
- Minimum ppm interval with predicted probabilities above 0.5, expected to be (-0.015, 0.015)

## How to apply

Download the pretrained pSCNN model and example NMR data from the Zenodo deposit (DOI 10.5281/zenodo.6504814) and set up Python 3.8.13 with TensorFlow 2.5.0-GPU via Anaconda (pip install tqdm) in the cloned pSCNN repository. Generate 980 varied spectral pairs by applying 35 chemical shift variations within +/-0.052 ppm, following the Table S6 shift scheme, to each of the 8 augmented mixture-component pairs (3 x 35 shifts x 8 pairs + variations = 980). Run 'pSCNN.py' to reload the pretrained model and feed the 980 varied spectral pairs for prediction. Record the predicted probability for every shift variation to pSCNN_shift_predictions.csv and determine the minimum ppm interval within which predicted probabilities stay above 0.5. The rationale is that translational invariance of CNNs should absorb chemical shift variation, so a wide probability-above-0.5 interval demonstrates robustness; the paper reports the interval (-0.015, 0.015) ppm, which exceeds the experimentally observed variation interval (-0.013, 0.010).

## Related tools

- **pSCNN** (Pretrained pseudo Siamese CNN reloaded to predict probabilities on the 980 varied spectral pairs) — https://github.com/yuxuanliao/pSCNN.git
- **Python 3.8.13** (Runtime environment for executing the shift-variation and prediction scripts) — https://github.com/yuxuanliao/pSCNN.git
- **TensorFlow 2.5.0-GPU** (Deep learning framework used to load and run the pretrained pSCNN model)
- **Anaconda** (Environment manager used to create the pSCNN Python 3.8.13 environment from requirements.txt) — https://www.anaconda.com/
- **tqdm** (Progress-bar dependency installed via pip for the prediction loop over varied spectral pairs)

## Examples

```
git clone https://github.com/yuxuanliao/pSCNN.git; conda activate pSCNN; python pSCNN.py
```

## Evaluation signals

- The total number of generated varied spectral pairs equals 980 (35 shift variations applied to 8 augmented mixture-component pairs)
- The minimum interval within which predicted probabilities remain above 0.5 is (-0.015, 0.015) ppm
- The tolerance interval (-0.015, 0.015) ppm exceeds the experimentally observed chemical shift variation interval (-0.013, 0.010) ppm
- pSCNN_shift_predictions.csv contains one predicted probability per shift variation per spectral pair
- pSCNN.py runs without error in the conda environment created from requirements.txt with TensorFlow 2.5.0-GPU

## Limitations

- Robustness was demonstrated only over the tested +/-0.052 ppm variation range using the Table S6 scheme; behavior outside this interval is unverified
- The evaluation uses augmented mixture-component pairs rather than the full experimental flavor mixtures, so real-world shift patterns may differ from the synthetic scheme
- No discussion-section content was available to extract additional limitations or reproducibility gaps

## Evidence

- [results] The pSCNN model maintains predicted probabilities above 0.5 over the chemical shift variation interval (−0.015, 0.015) ppm: "We have demonstrated that the translational invariance of convolutional neural networks can solve the chemical shift variation problem in NMR spectra."
- [methods] Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo.: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [methods] Download the model and these example data from Zenodo and reload pSCNN for prediction: "Download the model and these example data, pSCNN can be reload and predict easily."
- [readme] Environment setup in the pSCNN repository: "conda create --name pSCNN python=3.8.13
conda activate pSCNN
python -m pip install -r requirements.txt
pip install tqdm"
- [intro] Python 3.8.13 and TensorFlow (version 2.5.0-GPU): "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
