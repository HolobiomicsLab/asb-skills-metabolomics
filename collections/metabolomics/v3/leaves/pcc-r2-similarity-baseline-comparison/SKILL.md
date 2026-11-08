---
name: pcc-r2-similarity-baseline-comparison
description: Use when evaluating a learned ion-image representation for isotope or
  colocalized ion discovery in MSI and you need the established SIM-based baselines
  (Pearson correlation coefficient and R2) computed on identical query ions for direct
  comparison.
when_to_use_negative:
- When the goal is novelty detection on unseen molecules rather than reproducing the
  specific case-study isotope queries from this benchmark.
- When the MSI data are not preprocessed into the [X*Y, P] matrix and [P,1] peak-list
  format expected by DeepION and its similarity ranking pipeline.
- When only PCC/R2 values are needed without a learned representation to compare against
  — this skill is a baseline-comparison workflow, not a standalone PCC/R2 calculator.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: DeepION
  role: Provides the learned 20-dimensional ion image representations whose similarity
    rankings are benchmarked against the PCC and R2 baselines on the same query ions.
  repo: https://github.com/gankLei-X/DeepION
- name: rMSI
  role: Package whose isotope-ion identification metric (R2) is reproduced here as
    one of the two SIM-based baselines.
- name: METASPACE
  role: Web platform whose isotope-ion identification metric (PCC) is reproduced here
    as the other SIM-based baseline.
- name: PyTorch
  role: Framework used to run the DeepION encoder that produces the embeddings being
    compared against the baselines.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:214601b2866106343305f88eabe0991307322a818066c8450730ba385935f759
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - METASPACE
  - PyTorch
  - Python
  - Resnet18
  - rMSI
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pcc-r2-similarity-baseline-comparison

## Summary

Benchmark any ion-image similarity method (e.g. DeepION embeddings) against the PCC and R2 similarity metrics used for isotope/colocalized ion identification in METASPACE and the rMSI package, on the same rat brain MSI matrix.

## When to use

Use when evaluating a learned ion-image representation for isotope or colocalized ion discovery in MSI and you need the established SIM-based baselines (Pearson correlation coefficient and R2) computed on identical query ions for direct comparison.

## When NOT to use

- When the goal is novelty detection on unseen molecules rather than reproducing the specific case-study isotope queries from this benchmark.
- When the MSI data are not preprocessed into the [X*Y, P] matrix and [P,1] peak-list format expected by DeepION and its similarity ranking pipeline.
- When only PCC/R2 values are needed without a learned representation to compare against — this skill is a baseline-comparison workflow, not a standalone PCC/R2 calculator.

## Inputs

- Preprocessed MSI intensity matrix in [X*Y, P] format (e.g. DATASET/Pos_brain_data_matrix.txt, rat brain positive mode, 198 x 422 pixels)
- Preprocessed MSI peak list with one-dimensional shape [P,1] (e.g. DATASET/Pos_brain_data_peak.csv, 8422 ion peaks)
- Case-study query ion m/z values (e.g. 213.902, 214.047, 302.935, 699.493, 718.534, 1544.847, 1547.8702)
- 20-dimensional ion image representations from the method under test (e.g. DeepION embeddings, min-max scaled to [0,1])

## Outputs

- Per-query similarity rankings for each baseline (PCC, R2) and the method under test
- Comparison table (e.g. iso_case_study_results.csv) with ranking columns for DeepION, PCC, and R2 for every query ion
- Verified failure/success pattern per query (e.g. PCC < 0.5 and R2 < 0 for the m/z 1544.8471 / 1547.8702 pair)

## How to apply

Load the preprocessed MSI matrix as a 2D array of shape [X*Y, P] (e.g. the rat brain positive-mode matrix of 198 x 422 pixels with 8422 ion peaks from DATASET/Pos_brain_data_matrix.txt and Pos_brain_data_peak.csv). For each query ion from the case-study set (m/z 213.902, 214.047, 302.935, 699.493, 718.534, and the 1544.847/1547.8702 pair), compute PCC and R2 similarity across all other ion images, exactly as these metrics are used to identify isotope ions in METASPACE and the rMSI package, and rank candidates for each query. Run the same queries through the method under test (e.g. DeepION 20-dimensional UMAP-scaled embeddings) and assemble a per-query comparison table (iso_case_study_results.csv) with ranking columns for each method. Accept the baseline comparison when the similarity ranking reproduces the known failure modes: PCC < 0.5 and R2 < 0 for the m/z 1544.8471 / 1547.8702 isotope pair despite high missing-value proportions of 88.62% and 70.96%, while the learned embedding ranks the true partner highly.

## Related tools

- **DeepION** (Provides the learned 20-dimensional ion image representations whose similarity rankings are benchmarked against the PCC and R2 baselines on the same query ions.) — https://github.com/gankLei-X/DeepION
- **rMSI** (Package whose isotope-ion identification metric (R2) is reproduced here as one of the two SIM-based baselines.)
- **METASPACE** (Web platform whose isotope-ion identification metric (PCC) is reproduced here as the other SIM-based baseline.)
- **PyTorch** (Framework used to run the DeepION encoder that produces the embeddings being compared against the baselines.)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file Pos_ISO_result
```

## Evaluation signals

- The per-query comparison table contains ranking columns for DeepION, PCC, and R2 for every case-study query ion.
- For the m/z 1544.8471 / 1547.8702 pair, PCC is below 0.5 and R2 is below 0, while DeepION ranks the true isotope partner among the top neighbors.
- The reproduced ranking behaviour matches the results figures in the paper (e.g. DeepION correctly discovers the relevant isotope for queries 302.935, 699.493, 718.534, and 1544.847 in negative mode).
- Missing-value proportions for the failing isotope pair match the reported values (88.62% and 70.96%).

## Limitations

- PCC and R2 are pure signal-correlation metrics; they ignore the missing-value structure of ion images and therefore fail on low-SNR isotope pairs such as m/z 1544.847/1547.8702 (PCC < 0.5, R2 < 0 despite 88.62% and 70.96% missing values).
- The baselines require the same preprocessed [X*Y, P] matrix and [P,1] peak list as DeepION; they are not applicable to raw instrument output.
- The benchmark is defined on the specific case-study query ions of the rat brain MSI dataset used in the article; performance on other tissues or acquisition modes must be re-validated.

## Evidence

- [results] compared with two SIM-based methods, PCC and R2, which are used to identify isotope ions in MATASPACE30 and the rMSI package31: "compared with two SIM-based methods, PCC and R2, which are used to identify isotope ions in MATASPACE30 and the rMSI package31"
- [results] PCC < 0.5, R2 < 0 despite 88.62% and 70.96% missing values: [evidence span withheld — anchor status: not_found]
- [readme] python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file Pos_ISO_result: "python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file"
- [methods] The reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization.: [evidence span withheld — anchor status: not_found]
