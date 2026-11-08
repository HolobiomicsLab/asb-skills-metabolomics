---
name: chemical-shift-variation-simulation
description: Use when you need to verify that a spectral mixture-identification model
  is robust to chemical shift variation, by generating shifted spectral pairs and
  mapping the shift interval over which predictions stay above a threshold (here 0.5).
when_to_use_negative:
- The task is training or fine-tuning a model rather than stress-testing a pretrained
  model's robustness to chemical shift variation
- The spectra are not NMR spectra of small-molecule mixtures, so the ppm-scale chemical
  shift perturbation scheme is not meaningful
- You only need point-performance metrics (ACC/TPR/FPR) on an unchanged test set rather
  than a shift-sweep robustness interval
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: pSCNN
  role: Pretrained pseudo Siamese CNN reloaded via 'pSCNN.py' to predict mixture components
    on the chemically shifted spectral pairs
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: Python 3.8.13
  role: Runtime environment for generating shifted spectral pairs and running predictions
- name: TensorFlow 2.5.0-GPU
  role: Deep-learning backend used to execute the pretrained pSCNN model
- name: Anaconda
  role: Environment manager used to create the pSCNN Python 3.8.13 environment and
    install requirements.txt
  repo: https://www.anaconda.com/
- name: tqdm
  role: Progress-bar dependency installed alongside the main requirements for the
    prediction loop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:cf2507d0558e1fd33e145560ed81b0fd7fca5b5e3eb0d40d61b096f38fa2a09c
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# chemical-shift-variation-simulation

## Summary

Systematically perturb NMR spectral pairs with a grid of chemical shift variations and run the pretrained pSCNN model on the varied spectra to quantify how large a shift the model tolerates before its predicted probabilities drop below 0.5. This tests the translational-invariance claim of CNN-based compound identification in NMR mixtures.

## When to use

Use when you need to verify that a spectral mixture-identification model is robust to chemical shift variation, by generating shifted spectral pairs and mapping the shift interval over which predictions stay above a threshold (here 0.5).

## When NOT to use

- The task is training or fine-tuning a model rather than stress-testing a pretrained model's robustness to chemical shift variation
- The spectra are not NMR spectra of small-molecule mixtures, so the ppm-scale chemical shift perturbation scheme is not meaningful
- You only need point-performance metrics (ACC/TPR/FPR) on an unchanged test set rather than a shift-sweep robustness interval

## Inputs

- Pretrained pSCNN model and example NMR data from Zenodo DOI 10.5281/zenodo.6504814
- 8 augmented mixture-component NMR spectral pairs
- Chemical shift variation scheme (Table S6): 35 variations within +/-0.052 ppm

## Outputs

- 980 chemically shifted spectral pairs (35 variations x 8 pairs)
- Per-variation predicted probabilities for every shift variation (pSCNN_shift_predictions.csv)
- Minimum shift interval with predicted probabilities above 0.5, expected (-0.015, 0.015) ppm

## How to apply

Download the pretrained pSCNN model and example NMR data from the Zenodo deposit (DOI 10.5281/zenodo.6504814) and clone github.com/yuxuanliao/pSCNN. Set up a Python 3.8.13 environment with TensorFlow 2.5.0-GPU via Anaconda, installing the requirements.txt packages plus tqdm. Generate 980 varied spectral pairs by applying 35 chemical shift variations within +/-0.052 ppm (Table S6 scheme) to each of the 8 augmented mixture-component pairs. Run 'pSCNN.py' to reload the pretrained model and predict the 980 varied pairs, then record per-variation predicted probabilities and determine the minimum shift interval within which probabilities remain above 0.5. Save per-variation results to pSCNN_shift_predictions.csv and validate that the interval is (-0.015, 0.015) ppm, which exceeds the experimentally observed variation interval (-0.013, 0.010) ppm.

## Related tools

- **pSCNN** (Pretrained pseudo Siamese CNN reloaded via 'pSCNN.py' to predict mixture components on the chemically shifted spectral pairs) — https://github.com/yuxuanliao/pSCNN.git
- **Python 3.8.13** (Runtime environment for generating shifted spectral pairs and running predictions)
- **TensorFlow 2.5.0-GPU** (Deep-learning backend used to execute the pretrained pSCNN model)
- **Anaconda** (Environment manager used to create the pSCNN Python 3.8.13 environment and install requirements.txt) — https://www.anaconda.com/
- **tqdm** (Progress-bar dependency installed alongside the main requirements for the prediction loop)

## Examples

```
conda create --name pSCNN python=3.8.13; conda activate pSCNN; python -m pip install -r requirements.txt; pip install tqdm; python pSCNN.py
```

## Evaluation signals

- Prediction run covers all 980 varied pairs (35 shift variations x 8 augmented mixture-component pairs)
- Minimum interval with predicted probabilities above 0.5 equals (-0.015, 0.015) ppm
- The tolerance interval (-0.015, 0.015) ppm exceeds the experimentally observed variation interval (-0.013, 0.010) ppm
- pSCNN_shift_predictions.csv contains a predicted probability for every chemical shift variation applied
- Environment matches Python 3.8.13 with TensorFlow 2.5.0-GPU and tqdm installed

## Limitations

- Robustness is only demonstrated within the tested +/-0.052 ppm sweep; behavior outside this interval is uncharacterized
- The sweep uses 8 augmented mixture-component pairs, so coverage of mixture diversity is limited
- No discussion-section content was available, so limitations and reproducibility gaps were not extractable from the paper

## Evidence

- [supplementary] robustness interval finding: [evidence span withheld — anchor status: not_found]
- [abstract] translational invariance claim: "We have demonstrated that the translational invariance of convolutional neural networks can solve the chemical shift variation problem in NMR spectra."
- [methods] run pretrained model instructions: "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [readme] environment setup: "conda create --name pSCNN python=3.8.13
conda activate pSCNN
python -m pip install -r requirements.txt
pip install tqdm"
- [readme] zenodo deposit: "Since the model exceeded the limit, we have uploaded the model and all the NMR data to Zenodo."
