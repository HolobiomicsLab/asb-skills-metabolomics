---
name: gc-ci-hrms-isotope-labeling-data-visualization
description: Use when you have high-resolution GC-CI-MS autoQ integration results
  and need to visualize summed isotopologue distributions for a specific metabolite
  (e.g., Gly) across sample groups in a barplot.
when_to_use_negative:
- When the input is low-resolution (LRMS) data — sumIsotopologues is documented as
  only suitable for high-resolution autoQ results.
- When the goal is raw isotopologue table inspection rather than compound-specific
  barplot visualization.
- When the target compound is not present in the summed isotopologue table (metBarPlot
  will warn 'Not quantified').
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: isoSCAN
  role: Provides sumIsotopologues to aggregate autoQ isotope-integration results into
    per-metabolite isotopologue summary tables, and metBarPlot to render barplots
    of a single targeted compound (e.g., Gly) across sample groups.
  repo: https://github.com/jcapelladesto/isoSCAN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c02998
    title: Exploring the Use of Gas Chromatography Coupled to Chemical Ionization
      Mass Spectrometry (GC-CI-MS) for Stable Isotope Labeling in Metabolomics
  source_content_hash: sha256:c06eb36954a26abc5c38810d77602774b59322286ba27aa6560571bbe49576b8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jcapelladesto/isoSCAN
  tools:
  - R
  - isoSCAN
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# gc-ci-hrms-isotope-labeling-data-visualization

## Summary

Visualize summed isotopologue abundances from GC-CI-HRMS stable isotope labeling data using isoSCAN's metBarPlot, demonstrating the workflow for a targeted compound such as glycerol (Gly) across sample groups.

## When to use

Use when you have high-resolution GC-CI-MS autoQ integration results and need to visualize summed isotopologue distributions for a specific metabolite (e.g., Gly) across sample groups in a barplot.

## When NOT to use

- When the input is low-resolution (LRMS) data — sumIsotopologues is documented as only suitable for high-resolution autoQ results.
- When the goal is raw isotopologue table inspection rather than compound-specific barplot visualization.
- When the target compound is not present in the summed isotopologue table (metBarPlot will warn 'Not quantified').

## Inputs

- High-resolution autoQ integration table from isoSCAN processing of GC-CI-HRMS data
- Target compound name (e.g., 'Gly') from the summed isotopologue table

## Outputs

- Barplot of isotopologue abundances across sample groups (e.g., gly_metBarPlot.png)
- 'Not quantified' warnings for sample groups lacking quantified values

## How to apply

Apply the isoSCAN sumIsotopologues function to the autoQ integration table produced by high-resolution GC-CI-HRMS data processing. Extract rows for the target compound (e.g., CompoundName == 'Gly') from the resulting summed isotopologue table. Then call metBarPlot on these rows to generate a barplot of isotopologue abundances across sample groups, allowing 'Not quantified' warnings for groups lacking quantified values. Save the rendered figure (e.g., gly_metBarPlot.png) and verify the output file was created. This step is appropriate only for high-resolution data, as stated in the vignette.

## Related tools

- **isoSCAN** (Provides sumIsotopologues to aggregate autoQ isotope-integration results into per-metabolite isotopologue summary tables, and metBarPlot to render barplots of a single targeted compound (e.g., Gly) across sample groups.) — https://github.com/jcapelladesto/isoSCAN

## Examples

```
Rscript isoSCAN_vignette.R  # sumIsotopologues(autoQ_hr) ; gly <- subset(summed, CompoundName == 'Gly') ; metBarPlot(gly) ; png('gly_metBarPlot.png'); metBarPlot(gly); dev.off()
```

## Evaluation signals

- Verify the metBarPlot call completes without errors beyond the expected 'Not quantified' warnings for groups lacking quantified values.
- Confirm the rendered barplot file (e.g., gly_metBarPlot.png) exists and was created after the metBarPlot call.
- Check that the bars reflect the rows where CompoundName equals the target compound (e.g., 'Gly') from the summed isotopologue table.
- Confirm sumIsotopologues was applied to high-resolution autoQ integration results, not low-resolution data, as required by the vignette.
- Validate that sample groups lacking quantified values in the barplot correspond to those triggering 'Not quantified' warnings.

## Limitations

- sumIsotopologues (and thus the downstream barplot) is intended for high-resolution autoQ results only, per the vignette.
- metBarPlot emits 'Not quantified' warnings for sample groups without quantified values; these groups are still plotted but flagged.
- The visualization depends on the target compound being present in the summed isotopologue table; otherwise the output will be empty or missing.

## Evidence

- [methods] metBarPlot on Gly compound rows with Not quantified warnings: [evidence span withheld — anchor status: not_found]
- [results] sumIsotopologues on high-res autoQ data: [evidence span withheld — anchor status: not_found]
- [supplementary] Not quantified warnings in barplot: [evidence span withheld — anchor status: not_found]
- [discussion] vignette demo of barplot output: [evidence span withheld — anchor status: not_found]
- [intro] isoSCAN R-package for GC-CI-MS isotopologue quantification: [evidence span withheld — anchor status: not_found]
