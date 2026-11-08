---
name: prediction-probability-interval-analysis
description: Use when a pretrained classifier (e.g. pSCNN on NMR spectral pairs) must
  be stress-tested against a bounded input perturbation to determine the tolerable
  variation interval for confident (probability > 0.5) predictions.
when_to_use_negative:
- The goal is benchmarking classification accuracy (ACC/TPR/FPR) on fixed test sets
  rather than measuring robustness across a perturbation sweep
- Input spectra have no quantifiable, controllable perturbation axis (e.g. no chemical
  shift axis) to sweep systematically
- The perturbation range to test exceeds the model's validated operating range; results
  outside the tested +/-0.052 ppm scheme are not supported
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3474
tools:
- name: pSCNN
  role: Pretrained pseudo Siamese CNN reloaded via 'pSCNN.py' to predict mixture-component
    probabilities for each chemically shifted spectral pair
  repo: https://github.com/yuxuanliao/pSCNN.git
- name: Python 3.8.13
  role: Runtime environment for generating varied spectral pairs and running the prediction
    sweep
- name: TensorFlow 2.5.0-GPU
  role: Deep learning framework executing the pretrained pSCNN model for batch prediction
- name: Anaconda
  role: Environment manager used to create the pSCNN conda environment and install
    requirements
  repo: https://www.anaconda.com/
- name: tqdm
  role: Progress bar package installed alongside requirements for the prediction loop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/molecules27123653
    title: Deep Learning-Based Method for Compound Identification in NMR Spectra of
      Mixtures
  source_content_hash: sha256:0c579c1c3ac220fcf3843553d4e7ee9e1f46a3ffc109c76768081e36eba8beb4
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# prediction-probability-interval-analysis

## Summary

Quantifies a trained model's robustness to a controlled input perturbation (here, chemical shift variation in NMR mixture spectra) by sweeping perturbation magnitudes, recording predicted probabilities at each level, and identifying the minimum interval over which predictions stay above a 0.5 decision threshold.

## When to use

Use when a pretrained classifier (e.g. pSCNN on NMR spectral pairs) must be stress-tested against a bounded input perturbation to determine the tolerable variation interval for confident (probability > 0.5) predictions.

## When NOT to use

- The goal is benchmarking classification accuracy (ACC/TPR/FPR) on fixed test sets rather than measuring robustness across a perturbation sweep
- Input spectra have no quantifiable, controllable perturbation axis (e.g. no chemical shift axis) to sweep systematically
- The perturbation range to test exceeds the model's validated operating range; results outside the tested +/-0.052 ppm scheme are not supported

## Inputs

- Pretrained pSCNN model and example NMR data from Zenodo deposit 10.5281/zenodo.6504814
- 8 augmented mixture-component NMR spectral pairs
- Chemical shift variation scheme (Table S6): 35 variations within +/-0.052 ppm

## Outputs

- Per-variation predicted probabilities for all 980 spectral pairs (pSCNN_shift_predictions.csv)
- Minimum chemical shift interval with predicted probabilities above 0.5, expected as (-0.015, 0.015) ppm

## How to apply

Download the pretrained pSCNN model and example NMR data from the Zenodo deposit (DOI 10.5281/zenodo.6504814) and set up the Python 3.8.13 / TensorFlow 2.5.0-GPU environment (Anaconda, pip install tqdm) in the cloned repository. Generate the perturbed evaluation set by applying 35 chemical shift variations within +/-0.052 ppm (per Table S6 scheme) to each of the 8 augmented mixture-component pairs, yielding 980 varied spectral pairs. Run 'pSCNN.py' to reload the pretrained model and feed all 980 pairs for prediction. Record predicted probabilities for every shift variation to pSCNN_shift_predictions.csv, then determine the minimum shift interval within which probabilities remain above 0.5. Correct application is confirmed when that interval, (-0.015, 0.015) ppm, exceeds the experimentally observed chemical shift variation interval of (-0.013, 0.010) ppm.

## Related tools

- **pSCNN** (Pretrained pseudo Siamese CNN reloaded via 'pSCNN.py' to predict mixture-component probabilities for each chemically shifted spectral pair) — https://github.com/yuxuanliao/pSCNN.git
- **Python 3.8.13** (Runtime environment for generating varied spectral pairs and running the prediction sweep)
- **TensorFlow 2.5.0-GPU** (Deep learning framework executing the pretrained pSCNN model for batch prediction)
- **Anaconda** (Environment manager used to create the pSCNN conda environment and install requirements) — https://www.anaconda.com/
- **tqdm** (Progress bar package installed alongside requirements for the prediction loop)

## Examples

```
conda create --name pSCNN python=3.8.13 && conda activate pSCNN && python -m pip install -r requirements.txt && pip install tqdm && python pSCNN.py
```

## Evaluation signals

- 980 varied spectral pairs are generated (35 shift variations x 8 augmented mixture-component pairs)
- Every predicted probability across the sweep is recorded and saved to pSCNN_shift_predictions.csv
- The minimum interval with predicted probabilities above 0.5 equals (-0.015, 0.015) ppm
- The identified interval exceeds the experimentally observed chemical shift variation interval (-0.013, 0.010) ppm, matching the paper's finding

## Limitations

- Robustness is only validated within the tested +/-0.052 ppm shift scheme; behavior outside this range is unknown
- Results apply to the pSCNN architecture and the 8 augmented mixture-component pairs; other models or datasets are not covered
- No discussion-section content was available, so limitations and reproducibility gaps were not extracted (no changelog found)

## Evidence

- [results] The pSCNN model maintains predicted probabilities above 0.5 over the chemical shift variation interval (−0.015, 0.015) ppm: [evidence span withheld — anchor status: not_found]
- [methods] Run the file 'pSCNN.py': "Run the file 'pSCNN.py'. The model and these data have been uploaded at Zenodo."
- [intro] Python 3.8.13 and TensorFlow (version 2.5.0-GPU): "Python 3.8.13 and TensorFlow (version 2.5.0-GPU)"
- [readme] the model and all the NMR data to Zenodo: "Since the model exceeded the limit, we have uploaded the model and all the NMR data to Zenodo."
- [readme] Download the model and these example data, pSCNN can be reload and predict easily.: "Download the model and these example data, pSCNN can be reload and predict easily."
