---
name: proteomic-quantification-accuracy-evaluation
description: Use when comparing proteomics workflows (e.g., MaxQuant MBR vs DeepRTAlign
  alignment with MSFragger identification) on a benchmark mixture and you must show
  that identification gains preserve quantitative accuracy.
when_to_use_negative:
- Single-run LC-MS experiments with no retention time alignment across runs — there
  is no correspondence problem and no alignment-vs-quantification trade-off to evaluate
- Input is already an aligned, identified peptide intensity matrix — instead evaluate
  the upstream feature extraction, since the article notes alignment has less influence
  on quantification than feature extraction
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: DeepRTAlign
  role: Deep learning-based retention time alignment of MaxQuant feature lists across
    runs; provides the aligned feature groups whose peptide counts and quantification
    accuracy are evaluated
  repo: https://github.com/PHOENIXcenter/deeprtalign
- name: MaxQuant
  role: Feature extraction tool supplying allPeptides.txt feature lists and the original
    MBR baseline workflow
- name: MSFragger
  role: Provides peptide-spectrum identifications mapped onto aligned feature groups;
    its match-between-runs output serves as an alignment baseline
- name: OpenMS
  role: Feature alignment baseline against which aligned-peptide counts and quantification
    accuracy are compared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:f58b574a0a2949f8cc27c2cd20ef59ac7334ef1a6e039598845dcd8a167aca0d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - MSFragger
  - MaxQuant
  - OpenMS
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# proteomic-quantification-accuracy-evaluation

## Summary

Evaluates whether an LC-MS feature-alignment or identification workflow changes peptide quantification accuracy, by counting identified aligned peptides per workflow and comparing quantitative metrics across matched samples. Used in the DeepRTAlign Benchmark-FC study to show alignment gains do not compromise quantification.

## When to use

Use when comparing proteomics workflows (e.g., MaxQuant MBR vs DeepRTAlign alignment with MSFragger identification) on a benchmark mixture and you must show that identification gains preserve quantitative accuracy.

## When NOT to use

- Single-run LC-MS experiments with no retention time alignment across runs — there is no correspondence problem and no alignment-vs-quantification trade-off to evaluate
- Input is already an aligned, identified peptide intensity matrix — instead evaluate the upstream feature extraction, since the article notes alignment has less influence on quantification than feature extraction

## Inputs

- MaxQuant allPeptides.txt feature lists from Benchmark-FC HEK/E. coli mixture runs (delete the 'Resolution' column if MaxQuant version <2.0)
- Excel sample list mapping feature files to sample names (first column of allPeptides.txt)
- MSFragger peptide-spectrum identification (PSM) results
- OpenMS feature alignment and MSFragger MBR results for the same dataset

## Outputs

- Per-workflow counts of identified aligned peptides
- Quantification accuracy metrics per workflow
- benchmark_comparison.csv with counts and accuracy metrics

## How to apply

Run each candidate workflow on the same dataset — here, MaxQuant allPeptides.txt feature lists from the Benchmark-FC HEK/E. coli mixture aligned by DeepRTAlign (deeprtalign -m MaxQuant with an Excel sample list), with MSFragger peptide-spectrum identifications mapped onto aligned feature groups and QC at an FDR cutoff of 0.01. Build comparison baselines using the original MaxQuant MBR workflow, OpenMS feature alignment, and MSFragger match-between-runs on the same data. Count identified aligned peptides per workflow and compute quantification accuracy metrics on the same peptide groups across workflows. Correct application is judged by the alignment-driven workflow recovering substantially more peptides (reported as 150%, 7%, and 14.5% more than the three baselines) while quantification accuracy metrics remain comparable to baselines; write counts and accuracy metrics to benchmark_comparison.csv.

## Related tools

- **DeepRTAlign** (Deep learning-based retention time alignment of MaxQuant feature lists across runs; provides the aligned feature groups whose peptide counts and quantification accuracy are evaluated) — https://github.com/PHOENIXcenter/deeprtalign
- **MaxQuant** (Feature extraction tool supplying allPeptides.txt feature lists and the original MBR baseline workflow)
- **MSFragger** (Provides peptide-spectrum identifications mapped onto aligned feature groups; its match-between-runs output serves as an alignment baseline)
- **OpenMS** (Feature alignment baseline against which aligned-peptide counts and quantification accuracy are compared)

## Examples

```
deeprtalign -m MaxQuant -f file_dir -s sample_file.xlsx -pn 8
```

## Evaluation signals

- The DeepRTAlign-based workflow yields 150% more peptides than the original MaxQuant workflow, 7% more than OpenMS, and 14.5% more than MSFragger MBR on Benchmark-FC
- Quantification accuracy metrics of the alignment-driven workflow remain comparable to all baselines (no accuracy compromise)
- Aligned feature groups pass score-based quality control with FDR cutoff 0.01
- Counts and accuracy metrics are written to benchmark_comparison.csv with one row per workflow
- MaxQuant inputs satisfy format checks: sample list corresponds to the first column of allPeptides.txt and the 'Resolution' column is absent (or removed for MaxQuant <2.0)

## Limitations

- Alignment has limited influence on quantification compared with feature extraction; improving quantification accuracy requires optimizing feature extraction and alignment simultaneously (planned future work)
- When samples are quite different or many features share almost the same m/z, the default score-based QC reports very few results; then set FDR to 1 and select features with score > 0.5
- Do not run different projects under the same DeepRTAlign folder, as results will be overwritten

## Evidence

- [results] Benchmark-FC 150% more peptides: [evidence span withheld — anchor status: not_found]
- [discussion] alignment vs feature extraction: "since alignment has less influence on quantification than feature extraction in theory, we will try to improve the quantification accuracy by optimizing feature extraction and feature alignment"
- [methods] FDR QC default: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
- [readme] low-score fallback: "In this case, set FDR to 1, and select features with score greater than 0.5 are recomended."
- [discussion] alignment performance claim: "We have demonstrated that DeepRTAlign outperformed other existing alignment tools by aligning more corresponding features without compromising the quantification precision"
