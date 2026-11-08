---
name: metabolite-fold-change-class-counting
description: Use when metabolomics count data has KEGG compound hierarchies assigned
  and you need per-class (or per-subclass) tallies of compounds significantly changed
  between two groups, e.g. Streptomycin-treated vs Mock mouse fecal metabolites at
  sig_threshold 0.05.
when_to_use_negative:
- Compounds have no KEGG identifiers and you set keep_unknowns = FALSE, so most rows
  lack class metadata and per-class counts would be misleading
- You need per-compound statistics (fold change, p value, volcano plot) rather than
  category-level counts — omu_summary or plot_volcano is the appropriate step instead
- The comparison involves more than two groups or a multi-factor design where a two-group
  numerator/denominator omu_summary contrast does not apply
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: omu
  role: Provides assign_hierarchy (KEGG class metadata), omu_summary (t-test/fold
    change with p_adjust = "BH", test_type = "welch"), count_fold_changes (per-class
    counts at sig_threshold), and plot_bar (grouped bar plot)
  repo: github.com/connor-reid-tiffany/Omu
- name: ggplot2
  role: Rendering engine behind omu's plot_bar; the figure is a ggplot2 object compatible
    with ggplot2 themes
- name: KEGGREST
  role: Retrieves KEGG hierarchy/compound data underlying assign_hierarchy via keggGet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/mra.00129-19
    title: omu, a Metabolomics Count Data Analysis Tool for Intuitive Figures and
      Convenient Metadata Collection
  source_content_hash: sha256:94ee07c392285438ff02297feaa2d2080bf58fb9e5ce4b7b8ca37cecdae9995a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGGREST
  - R
  - ggplot2
  - omu
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# metabolite-fold-change-class-counting

## Summary

Count metabolites that significantly increased or decreased between two experimental groups, organized by KEGG compound class or subclass, using omu's count_fold_changes on omu_summary output joined with assign_hierarchy metadata, then render the counts as a grouped bar plot (Fig. 1A of the omu paper).

## When to use

Use when metabolomics count data has KEGG compound hierarchies assigned and you need per-class (or per-subclass) tallies of compounds significantly changed between two groups, e.g. Streptomycin-treated vs Mock mouse fecal metabolites at sig_threshold 0.05.

## When NOT to use

- Compounds have no KEGG identifiers and you set keep_unknowns = FALSE, so most rows lack class metadata and per-class counts would be misleading
- You need per-compound statistics (fold change, p value, volcano plot) rather than category-level counts — omu_summary or plot_volcano is the appropriate step instead
- The comparison involves more than two groups or a multi-factor design where a two-group numerator/denominator omu_summary contrast does not apply

## Inputs

- Metabolomics count data frame (e.g. c57_nos2KO_mouse_countDF) with KEGG-identified compounds
- Sample metadata file mapping compounds/samples to experimental factors (e.g. c57_nos2KO_mouse_metadata with a Treatment factor)
- KEGG compound hierarchy metadata from assign_hierarchy (Class/Subclass columns)

## Outputs

- Count table of metabolites significantly increased or decreased per KEGG class (or subclass)
- Grouped bar plot as a ggplot2 object saved as class_fold_change_barplot.png

## How to apply

Load the count data frame (e.g. the packaged example c57_nos2KO_mouse_countDF) and its metadata (c57_nos2KO_mouse_metadata), preferably via omu's read.metabo, then run assign_hierarchy with identifier = "KEGG" and keep_unknowns = TRUE so compounds lacking KEGG numbers are retained. Run omu_summary comparing the numerator group (e.g. "Strep") against the denominator (e.g. "Mock") on the Treatment factor with log_transform = TRUE, p_adjust = "BH" (Benjamini & Hochberg), and test_type = "welch" to get per-metabolite fold changes and adjusted p values. Apply count_fold_changes on the resulting statistics table with column = "Class" and sig_threshold = 0.05 to tally increased/decreased compounds per KEGG class. Finally, plot the count table with omu's plot_bar (ggplot2 object, fill colors dodgerblue2/firebrick2 with black outlines) and save it as class_fold_change_barplot.png.

## Related tools

- **omu** (Provides assign_hierarchy (KEGG class metadata), omu_summary (t-test/fold change with p_adjust = "BH", test_type = "welch"), count_fold_changes (per-class counts at sig_threshold), and plot_bar (grouped bar plot)) — github.com/connor-reid-tiffany/Omu
- **ggplot2** (Rendering engine behind omu's plot_bar; the figure is a ggplot2 object compatible with ggplot2 themes)
- **KEGGREST** (Retrieves KEGG hierarchy/compound data underlying assign_hierarchy via keggGet)

## Examples

```
count_data <- omu::assign_hierarchy(count_data, keep_unknowns = TRUE, identifier = "KEGG"); fc <- omu::count_fold_changes(omu::omu_summary(count_data, metadata, numerator = "Strep", denominator = "Mock", factor = "Treatment", log_transform = TRUE, p_adjust = "BH", test_type = "welch"), column = "Class", sig_threshold = 0.05)
```

## Evaluation signals

- class_fold_change_barplot.png exists as a PNG and is a valid ggplot2-rendered grouped bar plot
- Bar heights equal counts of compounds with adjusted p value below sig_threshold = 0.05 per class, split into Decrease and Increase
- Counts are computed only after assign_hierarchy with keep_unknowns = TRUE, so compounds without KEGG numbers are retained in the class totals
- Class-level summary is comparable to Fig. 1A of the omu paper for the Strep vs Mock contrast on the c57_nos2KO_mouse dataset
- The omu_summary run used p_adjust = "BH" and test_type = "welch" with log_transform = TRUE, i.e. BH-adjusted p values, not raw p values, drive the counts

## Limitations

- Only compounds with KEGG numbers receive class hierarchy unless keep_unknowns = TRUE; unknowns without classes cannot be tallied by Class
- The test_type argument is limited to "students", "welch", or "mwu", and the contrast is inherently two-group (numerator vs denominator)
- Counts depend on the chosen sig_threshold (0.05 here); the article notes bars above the sig_threshold value will be filled red, so threshold choice changes which bars are highlighted

## Evidence

- [abstract] provide a count table of every metabolite within a KEGG compound category that significantly increased or decreased between groups: "provide a count table of every metabolite within a KEGG compound category that significantly increased or decreased between groups"
- [abstract] we introduce omu, an R package designed for assigning compound hierarchies and linking compounds to corresponding enzyme and gene annotations: "we introduce omu, an R package designed for assigning compound hierarchies and linking compounds to corresponding enzyme and gene annotations"
- [methods] To assign hierarchical class data, use the ```assign_hierarchy``` function: "To assign hierarchical class data, use the ```assign_hierarchy``` function"
- [methods] a p value adjustment method of Benjamini & Hochberg with the argument ```p_adjust = "BH"```: "a p value adjustment method of Benjamini & Hochberg with the argument ```p_adjust = "BH"```"
- [methods] The figure is a ggplot2 object, so it is compatible with any ggplot2 themes: "The figure is a ggplot2 object, so it is compatible with any ggplot2 themes"
