---
name: retention-time-alignment-of-lc-ms-runs
description: Use when multiple mzML runs (e.g. a spike-in series with randomized technical
  triplicates on an Acquity/Synapt HDMS instrument) show retention-time drift that
  would break feature correspondence prior to linking feature maps.
when_to_use_negative:
- Input is already a linked feature table or an intensity matrix from TextExporter
  — alignment must precede feature linking
- Data consist of a single LC-MS run with no replicates to compare, so no cross-run
  drift exists to correct
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MapAlignmentPoseClustering
  role: Performs the retention time correction of feature maps between detection and
    linking steps
  repo: https://github.com/OpenMS/OpenMS
- name: OpenMS
  role: Framework providing MapAlignmentPoseClustering and the surrounding TOPP tool
    chain (version 1.10 in the paper)
  repo: https://github.com/OpenMS/OpenMS
- name: TOPPAS
  role: Graphical workflow editor used to embed RT correction in the fully automated
    pipeline
  repo: https://github.com/OpenMS/OpenMS
- name: KNIME
  role: Platform in which the RT-correction-containing workflow was implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1074/mcp.m113.031278
    title: Automated Label-free Quantification of Metabolites from Liquid Chromatography–Mass
      Spectrometry Data
  source_content_hash: sha256:4cda5e07cfec47a6a8599320e64e0329c0419706db33745d12d7e3b6752a6e6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - FeatureFinderMetabo
  - FeatureLinkerUnlabeledQT
  - KNIME
  - MapAlignmentPoseClustering
  - OpenMS
  - TOPPAS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Retention time alignment of LC-MS runs

## Summary

Corrects run-to-run chromatographic drift so that identical metabolite features eluting at slightly shifted retention times in different LC-MS injections can be linked across measurements. In the Kenar et al. 2014 label-free metabolomics workflow, this is done with OpenMS's MapAlignmentPoseClustering between feature detection (FeatureFinderMetabo) and feature linking (FeatureLinkerUnlabeledQT).

## When to use

Use when multiple mzML runs (e.g. a spike-in series with randomized technical triplicates on an Acquity/Synapt HDMS instrument) show retention-time drift that would break feature correspondence prior to linking feature maps.

## When NOT to use

- Input is already a linked feature table or an intensity matrix from TextExporter — alignment must precede feature linking
- Data consist of a single LC-MS run with no replicates to compare, so no cross-run drift exists to correct

## Inputs

- Feature maps (featureXML) produced by FeatureFinderMetabo from 33 mzML measurements of a human plasma spike-in series
- Reference feature map against which the other runs' retention-time axes are aligned

## Outputs

- Retention-time-corrected feature maps suitable for FeatureLinkerUnlabeledQT
- Consistently linked features whose intensities can be exported to a tabular text file for statistics

## How to apply

After detecting mass traces and assembling them into features with FeatureFinderMetabo (OpenMS 1.10), apply retention time correction to the feature maps with MapAlignmentPoseClustering. The algorithm matches high-confidence landmarks between maps and fits a pose-clustering transform that maps each run's retention-time axis onto a reference, compensating drift across triplicate injections. Alignment is a prerequisite for the subsequent FeatureLinkerUnlabeledQT step, which links coinciding (m/z, RT) features across all 33 measurements; without it, feature intensities for the same compound would not be joined correctly and quantification linearity (R >= 0.98 requirement) and replicate reproducibility would degrade. Align in the context of a fully automated TOPPAS/KNIME tool chain (feature detection -> RT correction -> feature linking -> TextExporter), adjusting parameters via the INIFileEditor where needed.

## Related tools

- **MapAlignmentPoseClustering** (Performs the retention time correction of feature maps between detection and linking steps) — https://github.com/OpenMS/OpenMS
- **OpenMS** (Framework providing MapAlignmentPoseClustering and the surrounding TOPP tool chain (version 1.10 in the paper)) — https://github.com/OpenMS/OpenMS
- **TOPPAS** (Graphical workflow editor used to embed RT correction in the fully automated pipeline) — https://github.com/OpenMS/OpenMS
- **KNIME** (Platform in which the RT-correction-containing workflow was implemented)

## Examples

```
MapAlignmentPoseClustering -in run1.featureXML run2.featureXML run3.featureXML -out aligned/
```

## Evaluation signals

- After alignment, FeatureLinkerUnlabeledQT links a consistent feature set (~1,168 features reproducible in >= 50% of the 33 measurements; 1,066 +/- 36 per run of 2,020 +/- 236 detected)
- Linked feature intensities for the seven spiked standards yield Pearson R >= 0.98 between concentration and intensity across detectable levels
- Triplicate feature intensities show little variation after the full pipeline including RT correction
- Alignment should not remove the true biological signal: the six lowest propionyl-l-carnitine-d3 levels remain non-detected, matching visual inspection

## Limitations

- Alignment quality depends on landmark quality from FeatureFinderMetabo; non-detected low-concentration spikes cannot be aligned by intensity signal
- Pose-clustering RT correction corrects global drift; paper gives no explicit threshold/parameter values for alignment tolerances
- The benchmark is specific to a Waters Acquity/Synapt HDMS plasma series; other separation technologies may need re-tuned parameters

## Evidence

- [results] This tool chain comprises feature detection, retention time correction, feature linking, and the ability to export to a tabular text file: "This tool chain comprises feature detection, retention time correction, feature linking, and the ability to export to a tabular text file"
- [results] MapAlignmentPoseClustering for retention time correction: "such as MapAlignmentPoseClustering for retention time correction"
- [results] FeatureLinkerUnlabeledQT for linking coinciding features from multiple experiments: "FeatureLinkerUnlabeledQT for linking coinciding features from multiple experiments"
- [readme] Over 150+ individual analysis tools (TOPP Tools), covering most MS and LC-MS data processing and mining tasks: [evidence span withheld — anchor status: external_doc]
