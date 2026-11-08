---
name: dinosaur-feature-extraction-dia
description: Use when you have raw DIA-MS runs (e.g. single-cell or bulk LC-MS cohorts)
  that need feature lists containing m/z, charge, RT and intensity columns before
  ID-free RT alignment with DeepRTAlign.
when_to_use_negative:
- Input is already an aligned feature table or identification-based peptide output
  rather than raw runs needing feature extraction
- Feature lists come from a format requiring special handling, e.g. MaxQuant allPeptides.txt
  (old versions with a 'Resolution' column), where Dinosaur mode is the wrong choice
  and MaxQuant-specific preprocessing is needed
- Samples are quite different or contain many features with nearly identical m/z,
  where the default score-based QC reports very few results and FDR=1 with a score>0.5
  cutoff is recommended instead
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Dinosaur
  role: Feature-detection tool that produces the per-run feature lists (m/z, charge,
    RT, intensity) used as DeepRTAlign input in Dinosaur mode
- name: DeepRTAlign
  role: Deep learning-based RT alignment tool that consumes the Dinosaur feature lists
    via '-m Dinosaur' and outputs the aligned feature table information_target.csv
  repo: https://github.com/PHOENIXcenter/deeprtalign
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43909-5
    title: 'DeepRTAlign: toward accurate retention time alignment for large cohort
      mass spectrometry data analysis'
  source_content_hash: sha256:aa79cb516b83c6575b84dbab5f0ee2b8281fd57c89f696e71533c9058a03598d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PHOENIXcenter/deeprtalign
  tools:
  - DeepRTAlign
  - Dinosaur
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# dinosaur-feature-extraction-dia

## Summary

Extract MS features (m/z, charge, RT, intensity) from DIA-MS runs using the Dinosaur feature-detection tool so that the resulting per-run feature lists can serve as direct input to DeepRTAlign's deep learning-based retention time alignment.

## When to use

Use when you have raw DIA-MS runs (e.g. single-cell or bulk LC-MS cohorts) that need feature lists containing m/z, charge, RT and intensity columns before ID-free RT alignment with DeepRTAlign.

## When NOT to use

- Input is already an aligned feature table or identification-based peptide output rather than raw runs needing feature extraction
- Feature lists come from a format requiring special handling, e.g. MaxQuant allPeptides.txt (old versions with a 'Resolution' column), where Dinosaur mode is the wrong choice and MaxQuant-specific preprocessing is needed
- Samples are quite different or contain many features with nearly identical m/z, where the default score-based QC reports very few results and FDR=1 with a score>0.5 cutoff is recommended instead

## Inputs

- Raw DIA-MS LC-MS runs (e.g. 18 HT-22 single-cell DIA-MS runs from PRIDE PXD025634)
- Dinosaur feature list files per run containing m/z, charge, RT and intensity columns
- Excel sample list mapping each Dinosaur feature file to its sample name

## Outputs

- Per-run Dinosaur feature lists (m/z, charge, RT, intensity)
- Post-QC aligned feature table information_target.csv in the mass_align_all_information folder with sample, fraction, time, mz, charge, intensity, score and group columns

## How to apply

Run Dinosaur on each DIA-MS run to produce per-run feature lists containing m/z, charge, RT and intensity columns; DeepRTAlign natively supports Dinosaur output via its '-m Dinosaur' method. Build an Excel sample list mapping each Dinosaur feature file to its sample name, and place the file_dir (containing the Dinosaur results) plus the sample file in a fresh working folder because DeepRTAlign overwrites results under a same folder. Invoke the alignment with 'deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx', setting the '-pn' parallel-processing parameter to the CPU core count to speed up the run; keep default parameters (1 min coarse-alignment time window, 0.03 m/z bin width, 2-decimal bin precision, score-based QC with FDR 0.01, keep_best_feature=1, keep_best_group=1). The rationale is that Dinosaur feature lists are one of the explicitly supported feature-extraction formats in DeepRTAlign's pipeline (preprocessing, coarse alignment, binning, filtering, alignment, generating result file), yielding the post-QC aligned feature table information_target.csv in the mass_align_all_information folder.

## Related tools

- **Dinosaur** (Feature-detection tool that produces the per-run feature lists (m/z, charge, RT, intensity) used as DeepRTAlign input in Dinosaur mode)
- **DeepRTAlign** (Deep learning-based RT alignment tool that consumes the Dinosaur feature lists via '-m Dinosaur' and outputs the aligned feature table information_target.csv) — https://github.com/PHOENIXcenter/deeprtalign

## Examples

```
deeprtalign -m Dinosaur -f file_dir -s sample_file.xlsx -pn <cpu_cores>
```

## Evaluation signals

- information_target.csv exists in the mass_align_all_information output folder with sample, fraction, time, mz, charge, intensity, score and group columns
- Aligned features share the same group name, allowing per-cell counts of aligned features/peptides present in at least two cells
- On the 18 HT-22 single-cell DIA cohort, per-cell aligned peptides (present in ≥2 cells) exceed the DIA-NN MBR reference by on average 39 peptides per cell (6.33%)
- The ratio of aligned features present in at least two cells to aligned peptides is approximately 42.3x
- QC-filtered rows have scores passing the default FDR cutoff of 0.01

## Limitations

- Do not run different projects under the same folder; DeepRTAlign results will be overwritten, so a new working folder is required
- When samples are quite different or many features share almost the same m/z, the default score-based QC usually reports very few results; setting FDR to 1 and selecting features with score > 0.5 is recommended
- time_window (default 1 min), bin_width (default 0.03) and bin_precision (default 2) depend on chromatography and feature-extraction parameters and may need adjustment
- A full Dinosaur-based DeepRTAlign run takes about 30 minutes on a normal computer without parallelization; use '-pn' per CPU core count for speed

## Evidence

- [readme] Feature lists are the output of feature extraction tools (DeepRTAlign supports Dinosaur, OpenMS, MaxQuant, XICFinder and any other text list containing m/z, charge, RT and Intensity information).: "Feature lists are the output of feature extraction tools (DeepRTAlign supports Dinosaur, OpenMS, MaxQuant, XICFinder and any other text list containing m/z, charge, RT and Intensity information )"
- [methods] step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file.: "step 1: preprocessing of input files, step 2: coarse alignment, step 3: bining, step 4: filtering, step 5: alignment, step 6: generating result file."
- [readme] The results will output to the mass_align_all_information folder.If you set kt to 0 (default), there will be only one file in the mass_align_all_information folder: information_target.csv: "The results will output to the mass_align_all_information folder.If you set kt to 0 (default), there will be only one file in the mass_align_all_information folder: information_target.csv (target"
- [abstract] we develop a deep learning-based RT alignment tool, DeepRTAlign, for large cohort LC-MS data analysis: "Here, we develop a deep learning-based RT alignment tool, DeepRTAlign, for large cohort LC-MS data analysis."
- [readme] fdr (float, 0-1, default 0.01) is the threshold used in quality control part.: "fdr (float, 0-1, default 0.01) is the threshold used in quality control part."
