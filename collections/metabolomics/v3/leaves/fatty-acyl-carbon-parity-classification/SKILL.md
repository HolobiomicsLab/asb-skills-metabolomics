---
name: fatty-acyl-carbon-parity-classification
description: Use when two TG annotation result sets (e.g., DIATAGeR FDR-filtered output
  vs MS-DIAL annotations on MS-DIAL alignment feature lists from the same DIA lipidomics
  run) must be compared by fatty-acyl carbon parity to quantify annotation completeness
  differences.
when_to_use_negative:
- The lipid class of interest is anything other than triacylglycerols — DIATAGeR explicitly
  states 'DIATAGeR is for triacylglycerols only' (lipid='TAG').
- The inputs are raw instrument files or spectra not exported from MS-DIAL/MZMine
  in the supported txt, msp, mgf, or mgf_mzmine formats; the pipeline expects MS-DIAL
  alignment feature lists or the generic feature-list format.
- The goal is a quantitative abundance comparison rather than identification/annotation
  completeness — this skill only counts fatty-acyl carbon parity among identifications,
  not TG signal intensities.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: DIATAGeR
  role: R package that produces the target set of FDR-filtered TG molecular-species
    annotations whose fatty-acyl compositions are parsed for even/odd carbon classification
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: MS-DIAL
  role: Source of the alignment feature lists (MSDIAL format) and the baseline TG
    annotation results against which DIATAGeR's even-carbon fatty-acyl counts are
    benchmarked
- name: FDRCutoff
  role: Final DIATAGeR step that filters combined target/decoy TG annotations to a
    pre-determined empirical FDR (cutoff=0.1, iteration=10) before parity classification
    is applied
  repo: https://github.com/Velenosi-Lab/DIATAGeR
- name: tidyverse
  role: R tooling used to parse TG fatty-acyl composition strings and compute the
    per-dataset, per-tool even-carbon percentages and differences
- name: data.table
  role: R tooling used for fast aggregation of fatty-acyl parity counts across the
    two tools' TG identification tables
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.aca.2025.344698
    title: 'DIATAGeR: Triacylglycerol annotation of data-independent acquisition based
      lipidomics'
  source_content_hash: sha256:cca531f1fbd5885e375ff18f8bcf2d7c63ee3244562f5ae382b666e52fde19d4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Velenosi-Lab/DIATAGeR
  tools:
  - DIATAGeR
  - FDRCutoff
  - LipidIdentifier
  - MS-DIAL
  - RemoveIsotopes
  - ScoreLipids
  - SpectraImport
  - data.table
  - measureFileImport
  - tidyverse
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 20
---

# fatty-acyl-carbon-parity-classification

## Summary

Classifies the fatty-acyl composition of annotated triacylglycerol (TG) identifications into even- vs odd-carbon chains and compares the even-carbon fractions between two annotation pipelines (e.g., DIATAGeR vs MS-DIAL) per tissue dataset.

## When to use

Use when two TG annotation result sets (e.g., DIATAGeR FDR-filtered output vs MS-DIAL annotations on MS-DIAL alignment feature lists from the same DIA lipidomics run) must be compared by fatty-acyl carbon parity to quantify annotation completeness differences.

## When NOT to use

- The lipid class of interest is anything other than triacylglycerols — DIATAGeR explicitly states 'DIATAGeR is for triacylglycerols only' (lipid='TAG').
- The inputs are raw instrument files or spectra not exported from MS-DIAL/MZMine in the supported txt, msp, mgf, or mgf_mzmine formats; the pipeline expects MS-DIAL alignment feature lists or the generic feature-list format.
- The goal is a quantitative abundance comparison rather than identification/annotation completeness — this skill only counts fatty-acyl carbon parity among identifications, not TG signal intensities.

## Inputs

- MS-DIAL alignment feature lists (e.g., Feature_List_Liver_MSDIAL.txt and the corresponding plasma file) from MetaboLights deposit MTBLS11927
- Centroided MS2 spectra in txt format exported from MS-DIAL (e.g., Spectra_txt_Liver_MSDIAL/Centroid folder)
- DIATAGeR FDR-filtered TG identification results (ResultsTG_FDR RDS object or equivalent output file)
- MS-DIAL TG annotation results on the same datasets

## Outputs

- Per-dataset, per-tool counts and percentages of even-carbon fatty acyls among annotated TGs
- DIATAGeR-over-MS-DIAL difference in even-carbon fatty-acyl percentage for liver and plasma
- Summary CSV table (dataset, tool, n TGs, % even-carbon fatty acyls, difference), e.g., msdial_benchmark_summary.csv

## How to apply

Run each annotation pipeline end-to-end first: for DIATAGeR, import the MS-DIAL alignment feature list with measureFileImport (IonMode='Pos', ResultsFiletype='MSDIAL'), attach MS2 spectra via SpectraImport (spectra.file.type='txt', rttol=10), clean spectra with RemoveIsotopes (ppmtol=15, low.intensity.cutoff=100, density.cutoff=TRUE, density.cutoff.value=1), identify with LipidIdentifier (lipid='TAG', ppmtol=15, rttol=5, ms1.precursors=1, ms2.precursors=1, default max.tails), score with ScoreLipids, and filter with FDRCutoff (cutoff=0.1, iteration=10). Then parse the fatty-acyl composition string of each annotated TG from both DIATAGeR and MS-DIAL outputs, labeling each fatty acyl as even- or odd-carbon. Compute the percentage of even-carbon fatty acyls among TGs for each tool and dataset (liver vs plasma) separately, and report the DIATAGeR-over-MS-DIAL difference, which the source study reports as +18% (liver) and +12% (plasma). Write a summary table with columns dataset, tool, n TGs, % even-carbon fatty acyls, and difference (e.g., msdial_benchmark_summary.csv) and check the recomputed differences against the reported values.

## Related tools

- **DIATAGeR** (R package that produces the target set of FDR-filtered TG molecular-species annotations whose fatty-acyl compositions are parsed for even/odd carbon classification) — https://github.com/Velenosi-Lab/DIATAGeR
- **MS-DIAL** (Source of the alignment feature lists (MSDIAL format) and the baseline TG annotation results against which DIATAGeR's even-carbon fatty-acyl counts are benchmarked)
- **FDRCutoff** (Final DIATAGeR step that filters combined target/decoy TG annotations to a pre-determined empirical FDR (cutoff=0.1, iteration=10) before parity classification is applied) — https://github.com/Velenosi-Lab/DIATAGeR
- **tidyverse** (R tooling used to parse TG fatty-acyl composition strings and compute the per-dataset, per-tool even-carbon percentages and differences)
- **data.table** (R tooling used for fast aggregation of fatty-acyl parity counts across the two tools' TG identification tables)

## Examples

```
ResultsTG_FDR <- FDRCutoff(identified.lipids = ResultsTG, iteration = 10, cutoff = 0.1, rttol = 10)
```

## Evaluation signals

- Recomputed DIATAGeR-over-MS-DIAL differences reproduce the reported values: 18% more even-carbon fatty acyls in liver and 12% more in plasma, with any discrepancy recorded.
- The summary table (dataset, tool, n TGs, % even-carbon fatty acyls, difference) is complete for both liver and plasma datasets and both tools, with no missing parity assignments.
- Every parsed fatty acyl from every annotated TG is classified as exactly even- or odd-carbon (exhaustive binary partition of all acyl chains).
- DIATAGeR identifications entering the comparison passed FDRCutoff with cutoff=0.1 and iteration=10, and were produced with LipidIdentifier settings ppmtol=15, rttol=5, ms1.precursors=1, ms2.precursors=1 and the default max.tails list (8.2 through 26.0).
- Both tools were applied to identical input feature lists from the same MetaboLights deposit (MTBLS11927), so the comparison is apples-to-apples.

## Limitations

- DIATAGeR is restricted to triacylglycerol annotation; the source explicitly states 'DIATAGeR is for triacylglycerols only', so parity comparisons are limited to TG identifications.
- Parity classification depends on correctly parsing the fatty-acyl composition embedded in each tool's annotation strings; divergent annotation formats between MS-DIAL and DIATAGeR can cause parse errors or misclassification.
- The article index reports missing discussion-section content and no changelog, so reproducibility caveats and parameter change history are not documented in the extracted source.
- Features with missing spectra are diverted to a 'Troubleshooting' folder during SpectraImport, so some features may silently drop out of the comparison if not checked.

## Evidence

- [methods] classifying each fatty acyl as even- or odd-carbon: [evidence span withheld — anchor status: not_found]
- [results] reported benchmark deltas 18% and 12%: [evidence span withheld — anchor status: not_found]
