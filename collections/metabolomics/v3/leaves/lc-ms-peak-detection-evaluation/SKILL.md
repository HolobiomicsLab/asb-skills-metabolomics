---
name: lc-ms-peak-detection-evaluation
description: Use when evaluating or reproducing peak-detection accuracy on Orbitrap/QTOF
  LC-MS raw files against a manually labeled EIC ground-truth set, or comparing MassCube
  with MS-DIAL, MZmine3, and xcms.
when_to_use_negative:
- The input is already a processed feature table rather than raw LC-MS data — peak
  detection cannot be re-evaluated from feature tables.
- You need metabolite identification or MS/MS annotation benchmarking; MassCube does
  not directly perform metabolite identification.
- The study requires a graphical user interface — no GUI is available in the current
  MassCube version.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MassCube
  role: 'Peak detection engine: constructs mass traces via signal clustering and Gaussian-filter-assisted
    edge detection, then groups peaks for adduct/ISF detection'
  repo: https://github.com/huaxuyu/masscube
- name: Python
  role: Runtime environment; MassCube is installed via pip and driven through its
    Python API
  repo: https://github.com/huaxuyu/masscube/
- name: MS-DIAL
  role: Comparison tool re-run on the same labeled EIC set to benchmark detection
    accuracy
- name: MZmine3
  role: Comparison tool re-run on the same labeled EIC set to benchmark detection
    accuracy
- name: XCMS
  role: Comparison tool re-run on the same labeled EIC set to benchmark detection
    accuracy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-60640-5
    title: MassCube improves accuracy for metabolomics data processing from raw files
      to phenotype classifiers
  source_content_hash: sha256:ff7e0f71722f5dfe8b214acb0e83e3c3f40c6872fdf03cc9d5d4f16a977c05fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/huaxuyu/masscube
  tools:
  - MS-DIAL
  - MZmine3
  - MassCube
  - Python
  - XCMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# lc-ms-peak-detection-evaluation

## Summary

Benchmark LC-MS peak-detection accuracy of MassCube against MS-DIAL, MZmine3, and XCMS using manually labeled extracted ion chromatograms (EICs), scoring double-peak and single-peak recognition separately.

## When to use

Use when evaluating or reproducing peak-detection accuracy on Orbitrap/QTOF LC-MS raw files against a manually labeled EIC ground-truth set, or comparing MassCube with MS-DIAL, MZmine3, and xcms.

## When NOT to use

- The input is already a processed feature table rather than raw LC-MS data — peak detection cannot be re-evaluated from feature tables.
- You need metabolite identification or MS/MS annotation benchmarking; MassCube does not directly perform metabolite identification.
- The study requires a graphical user interface — no GUI is available in the current MassCube version.

## Inputs

- Experimental Orbitrap/QTOF LC-MS raw data files (mzML or supported Bruker .d inputs)
- Zenodo record 14159704: 722 manually labeled EIC PNG images
- Metadata list mapping each labeled EIC image to its ground-truth peak-shape class (double peak vs single peak)

## Outputs

- peak_detection_accuracy.csv with double-peak accuracy, single-peak accuracy, and per-image agreement flags
- Per-category accuracy scores (reference: 93.5% double peaks, 97.3% single peaks)

## How to apply

Install MassCube from PyPI with 'pip install masscube' and download Zenodo record 14159704, which contains 722 pre-labeled EIC PNG images plus a metadata list recording the ground-truth double-peak/single-peak class for each. Run the MassCube untargeted workflow — peak detection by mass-trace construction with signal clustering and Gaussian-filter-assisted edge detection, followed by peak grouping for adduct and in-source fragment detection — on the associated experimental Orbitrap/QTOF raw files so every labeled EIC is covered by a MassCube-detected peak. Match each detected peak back to its labeled EIC via the metadata list and assign the software-predicted double-peak or single-peak class. Compute per-category accuracy as the fraction of labeled EICs agreeing with ground truth; the reported references are 93.5% for double peaks and 97.3% for single peaks. Optionally rerun the same matching on MS-DIAL, MZmine3, and xcms outputs to reproduce the comparative benchmark, writing per-image agreement flags to peak_detection_accuracy.csv.

## Related tools

- **MassCube** (Peak detection engine: constructs mass traces via signal clustering and Gaussian-filter-assisted edge detection, then groups peaks for adduct/ISF detection) — https://github.com/huaxuyu/masscube
- **Python** (Runtime environment; MassCube is installed via pip and driven through its Python API) — https://github.com/huaxuyu/masscube/
- **MS-DIAL** (Comparison tool re-run on the same labeled EIC set to benchmark detection accuracy)
- **MZmine3** (Comparison tool re-run on the same labeled EIC set to benchmark detection accuracy)
- **XCMS** (Comparison tool re-run on the same labeled EIC set to benchmark detection accuracy)

## Examples

```
pip install masscube
```

## Evaluation signals

- Double-peak accuracy on the 722 labeled EICs is ~93.5% for MassCube, exceeding MS-DIAL, MZmine3, and xcms.
- Single-peak accuracy is ~97.3% for MassCube, exceeding the three comparison tools.
- Every one of the 722 labeled EICs is matched to a MassCube-detected peak (100% signal coverage reported by the authors).
- peak_detection_accuracy.csv exists and contains a binary agreement flag for each of the 722 labeled images plus the two per-category accuracy scores.

## Limitations

- Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all peak-shape scenarios.
- Convolution in experimental data (varying scan numbers across peaks, peak tailing) is difficult to explore systematically with synthetic data, so the labeled-EIC benchmark is the practical validation route.
- External validated benchmarking datasets with large-scale ground-truth data remain scarce beyond this labeled set.

## Evidence

- [results] MassCube achieved the highest accuracy for double peaks (93.5%) and single peaks (97.3%), outperforming MS-DIAL, MZmine3, and xcms on the 722 manually labeled extracted ion chromatograms.: [evidence span withheld — anchor status: not_found]
- [abstract] Peak detection method description: "peaks are detected by constructing mass traces through signal clustering and Gaussian-filter assisted edge detection"
- [abstract] Peak detection coverage claim: "Peak detection in MassCube achieves 100% signal coverage with comprehensive reporting of chromatographic metadata"
- [discussion] Manual peak-shape rules limitation: "Human-defined rules for peak shapes, reflected in algorithmic parameters, cannot account for all such scenarios."
- [readme] Installation instructions: "To install **masscube**, open a terminal and run:

```bash
pip install masscube
```"
