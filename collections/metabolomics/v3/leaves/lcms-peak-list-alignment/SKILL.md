---
name: lcms-peak-list-alignment
description: Use when a set of peak lists from multiple LC-MS samples (e.g., 20 synthetic
  lipidomic samples derived from a seed sample) must be merged into an aligned peak
  list, especially when random non-linear RT shifts are present.
when_to_use_negative:
- Input is already an aligned peak list or feature table — re-alignment would not
  add information and may degrade ground-truth correspondences
- The task is raw data processing (peak picking, deisotoping, gap filling, normalization)
  rather than aligning existing peak lists; run those MZmine 2 modules first
- Data types outside LC-MS scope of the tested modules — the article notes developed
  modules were tested mainly on LC-MS data only
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine 2
  role: Provides the RANSAC peak list alignment module and Join aligner used to align
    peak lists at RT tolerances of 5 s, 20 s, and 50 s
  repo: https://github.com/mzmine/mzmine2
- name: Java
  role: Implementation/build platform; MZmine 2 is platform independent and built
    with ./gradlew or gradlew.bat
  repo: https://github.com/mzmine/mzmine2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-11-395
    title: 'MZmine 2: Modular framework for processing, visualizing, and analyzing
      mass spectrometry-based molecular profile data'
  source_content_hash: sha256:5ce54b5536333a37bec3d1e363ed2eb5167dc67927e1339ed2f34991fd589034
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine2
  tools:
  - Java
  - MZmine 2
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# lcms-peak-list-alignment

## Summary

Aligns peak lists across LC-MS samples so that the same metabolite feature appears in one row, using MZmine 2's RANSAC-based peak list alignment — a random sample consensus method robust to non-linear retention time deviation — evaluated by precision and recall against ground-truth alignments.

## When to use

Use when a set of peak lists from multiple LC-MS samples (e.g., 20 synthetic lipidomic samples derived from a seed sample) must be merged into an aligned peak list, especially when random non-linear RT shifts are present.

## When NOT to use

- Input is already an aligned peak list or feature table — re-alignment would not add information and may degrade ground-truth correspondences
- The task is raw data processing (peak picking, deisotoping, gap filling, normalization) rather than aligning existing peak lists; run those MZmine 2 modules first
- Data types outside LC-MS scope of the tested modules — the article notes developed modules were tested mainly on LC-MS data only

## Inputs

- Peak lists from 20-sample lipidomic synthetic datasets (12 datasets derived from seed samples with random non-linear retention time deviation)
- Ground-truth alignment tables (e.g., 'ground truth' alignments from Lange et al. for P1, P2, M1, M2 proteomic/metabolomic datasets)
- Raw LC-MS data files for upstream import into MZmine 2

## Outputs

- Aligned peak lists (rows of correspondences across samples)
- Precision and recall values per dataset and per RT tolerance (e.g., ransac_precision_recall_results.csv covering 12 datasets x 3 tolerances)

## How to apply

Import the peak lists into MZmine 2 and run the RANSAC peak list alignment module, sweeping retention-time tolerances such as 5 s, 20 s, and 50 s; MZmine 2 was developed in Java and is platform independent. Compare each aligned peak list against known ground-truth correspondences (e.g., synthetic ground truth from Lange et al. datasets P1, P2, M1, M2) to compute precision and recall per dataset and tolerance. Rationale: in the source study, only the RANSAC algorithm achieved 100% in both precision and recall on 12 synthetic lipidomic datasets (20 samples each, built from one seed sample per study with random non-linear RT deviation), while the Join aligner did not reach perfect alignment at the tested tolerances; RANSAC provided better results in 11 of 13 alignments against experimental data. Aggregate per-dataset and per-tolerance precision/recall into a results table (e.g., ransac_precision_recall_results.csv) for comparison across aligners.

## Related tools

- **MZmine 2** (Provides the RANSAC peak list alignment module and Join aligner used to align peak lists at RT tolerances of 5 s, 20 s, and 50 s) — https://github.com/mzmine/mzmine2
- **Java** (Implementation/build platform; MZmine 2 is platform independent and built with ./gradlew or gradlew.bat) — https://github.com/mzmine/mzmine2

## Examples

```
./gradlew && java -jar startMZmine.jar  # build MZmine 2, then run Peak list methods > RANSAC peak list aligner with MZ/RT tolerance; sweep RT tolerance 5 s, 20 s, 50 s
```

## Evaluation signals

- On the 12 synthetic lipidomic datasets, the RANSAC aligner must reproduce 100% precision AND 100% recall while the Join aligner does not reach perfect alignment at 5 s, 20 s, or 50 s RT tolerance
- Precision/recall computed against known ground-truth correspondences must be reported per dataset and per tolerance (12 datasets x 3 tolerances)
- The results CSV must contain all 36 dataset-tolerance combinations without missing values
- On experimental benchmarks (P1, P2, M1, M2), RANSAC should outperform Join aligner in 11 of 13 alignments; a worse result is expected only for P2 dataset fraction 00, which has over 6800 rows after alignment

## Limitations

- The Join aligner may be preferable in some cases — RANSAC was worse in a single case (P2 dataset fraction 00, over 6800 rows after alignment)
- The 100% precision/recall result was demonstrated on synthetic datasets; performance on real experimental data is benchmark-based (11 of 13 alignments better) rather than perfect
- The published data hash for ProteomeCommons.org Tranche download is absent from the manuscript text, so reproducing the exact synthetic datasets may require contacting authors or regenerating data
- Modules were tested mainly on LC-MS data; validation on GC-MS or other data types was not demonstrated

## Evidence

- [results] RANSAC achieves 100% precision/recall on synthetic data: "Only the use of the RANSAC algorithm achieved 100% in both precision and recall performance on these synthetic data sets."
- [results] Synthetic dataset construction: "12 synthetic datasets were created using samples from 12 different lipidomic studies. A single sample from each study was used as a seed to create a synthetic set of 20 samples"
- [results] RANSAC vs Join aligner on experimental data: "the RANSAC aligner provided better results in 11 of 13 alignments, with worse results obtained in only a single case (P2 dataset fraction 00)"
- [intro] Alignment is a new MZmine 2 method: "We also introduce a new and efficient method for peak list alignment that was implemented in MZmine 2."
- [supplementary] Supplementary precision/recall tables: "Precision and recall values of RANSAC and Join aligner results for 12 synthetic data sets."
- [readme] Java-based, platform independent: "mzmine is an open-source software for mass spectrometry data processing. The goals of the project is to provide a user-friendly, flexible and easily extendable software with a complete set of modules"
