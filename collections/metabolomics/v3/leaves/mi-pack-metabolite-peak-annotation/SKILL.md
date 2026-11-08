---
name: mi-pack-metabolite-peak-annotation
description: Use when a processed, filtered and imputed LC-MS or DIMS feature table
  (post XCMS-Basic-Birmingham-Pipe, Replicate/Blank/Sample Filtering and missing value
  imputation) needs putative metabolite annotations attached to its peaks.
when_to_use_negative:
- Input is raw vendor LC-MS/DIMS spectra rather than a processed, filtered peak list
  — MI-Pack runs downstream of XCMS feature detection/alignment and the filtering/imputation
  tools
- The goal is statistical separation testing between sample groups rather than metabolite
  identification — use the Galaxy-M PLS-Toolbox PCA with Scores Test instead
- Only preliminary feature-group annotation (e.g. adduct/isotope grouping) is needed
  and MI-Pack's formula+pathway identification is out of scope — the paper also lists
  Camera and PUTMEDID as identification software
tools:
- name: MI-Pack
  role: 'Performs the annotation itself: empirical formula search on processed peak
    masses and assignment of putative metabolite identifications by integrating accurate
    masses and metabolic pathways'
  repo: https://github.com/Viant-Metabolomics/MI-Pack
- name: Parallel Python
  role: Optional parallelization of MI-Pack's empirical formula search to speed up
    annotation
- name: Galaxy-M
  role: Wraps MI-Pack as an annotation tool within the end-to-end Galaxy metabolomics
    workflow (step 6 of the LC-MS history)
  repo: https://github.com/Viant-Metabolomics/Galaxy-M
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13742-016-0115-8
    title: 'Galaxy-M: a Galaxy workflow for processing and analyzing direct infusion
      and liquid chromatography mass spectrometry-based metabolomics data'
  source_content_hash: sha256:7699a6e86d1188c2e1f8213210ee48c776b68eedadda634565f3be9bc88e6cca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Viant-Metabolomics/MI-Pack
  tools:
  - Galaxy-M
  - MI-Pack
  - PLS-Toolbox
  - Parallel Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mi-pack-metabolite-peak-annotation

## Summary

Assign putative metabolite identifications to processed mass-spectral peaks using MI-Pack, which integrates accurate masses with metabolic pathways and empirical formula search against compound reference databases (KEGG, HMDB, LIPID MAPS). It is the annotation step of the Galaxy-M LC-MS and DIMS workflows.

## When to use

Use when a processed, filtered and imputed LC-MS or DIMS feature table (post XCMS-Basic-Birmingham-Pipe, Replicate/Blank/Sample Filtering and missing value imputation) needs putative metabolite annotations attached to its peaks.

## When NOT to use

- Input is raw vendor LC-MS/DIMS spectra rather than a processed, filtered peak list — MI-Pack runs downstream of XCMS feature detection/alignment and the filtering/imputation tools
- The goal is statistical separation testing between sample groups rather than metabolite identification — use the Galaxy-M PLS-Toolbox PCA with Scores Test instead
- Only preliminary feature-group annotation (e.g. adduct/isotope grouping) is needed and MI-Pack's formula+pathway identification is out of scope — the paper also lists Camera and PUTMEDID as identification software

## Inputs

- Processed and filtered LC-MS feature/peak table from the Galaxy-M pipeline (XCMS-Basic-Birmingham-Pipe output after Replicate Filtering, Blank Filtering, Sample Filtering, imputation and normalization)
- MI-Pack Python package (GitHub commit 06ce0ace643ee1cf1d27550769a5272b7ea50825)
- Compound reference databases (e.g. KEGG, HMDB, LIPID MAPS)

## Outputs

- Annotated peak table assigning putative metabolite identifications (empirical formulas plus pathway-integrated IDs) to the processed peaks
- MI-Pack annotation output consistent with the published Galaxy-M workflow history

## How to apply

Run MI-Pack as the final annotation stage of the Galaxy-M workflow, after peak picking/alignment (XCMS with CAMERA), the three filtering tools, and missing value imputation/normalization have produced the processed peak list. MI-Pack performs an empirical formula search on the accurate masses of the processed peaks and then assigns putative metabolite identifications by integrating those masses with metabolic pathway knowledge, drawing on compound reference databases such as KEGG, HMDB and LIPID MAPS. Because the empirical formula search can be expensive, optionally enable parallel processing via Parallel Python (version 1.6.4) to speed it up. Install MI-Pack as a Python package from its GitHub repository (python setup.py build; python setup.py install); the GigaScience publication pinned MI-Pack to GitHub commit 06ce0ace643ee1cf1d27550769a5272b7ea50825. The output is an annotation table over the processed peaks that should be checked against the published Galaxy-M history for the case study being reproduced.

## Related tools

- **MI-Pack** (Performs the annotation itself: empirical formula search on processed peak masses and assignment of putative metabolite identifications by integrating accurate masses and metabolic pathways) — https://github.com/Viant-Metabolomics/MI-Pack
- **Parallel Python** (Optional parallelization of MI-Pack's empirical formula search to speed up annotation)
- **Galaxy-M** (Wraps MI-Pack as an annotation tool within the end-to-end Galaxy metabolomics workflow (step 6 of the LC-MS history)) — https://github.com/Viant-Metabolomics/Galaxy-M

## Examples

```
python setup.py build && python setup.py install  # MI-Pack install from https://github.com/Viant-Metabolomics/MI-Pack
```

## Evaluation signals

- An annotation output is generated for every processed peak in the filtered feature table, not just a subset
- Annotations are consistent with the published Galaxy-M workflow history for the case study (e.g. the MTBLS146 LC-MS reproduction validated that the MI-Pack annotation output was generated and matched the published history)
- Each assignment rests on both an empirical formula match to the accurate mass and supporting metabolic pathway evidence (MI-Pack's stated confidence mechanism)
- Running with and without Parallel Python produces the same annotations (parallelization affects speed only)
- Identifications trace back to the compound reference databases used (KEGG, HMDB, LIPID MAPS)

## Limitations

- Identifications are putative — MI-Pack's design goal is increased confidence by combining accurate masses with metabolic pathways, not definitive structural confirmation
- Requires Python 2.7 (the version used for Galaxy-M development); Parallel Python 1.6.4 is non-essential, so without it the empirical formula search may be slow
- The publication depends on a pinned MI-Pack GitHub commit (06ce0ace643ee1cf1d27550769a5272b7ea50825); other versions are untested in this workflow
- MI-Pack is GPLv3 licensed, with no restrictions on use by non-academics, but outputs depend on the quality/completeness of the reference databases

## Evidence

- [abstract] Metabolite annotation is performed using MI-Pack [10].: "Metabolite annotation is performed using MI-Pack [10]."
- [methods] Download the MI-Pack python package from https://github.com/Viant-Metabolomics/MI-Pack: "Download the MI-Pack python package from https://github.com/Viant-Metabolomics/MI-Pack"
- [methods] parallel processing to speed up e.g. Empirical Formula search: "parallel processing to speed up e.g. Empirical Formula search"
- [readme] MI-Pack: Increased confidence of metabolite identification in mass spectra by integrating accurate masses and metabolic pathways: "MI-Pack: Increased confidence of metabolite identification in mass spectra by integrating accurate masses and metabolic pathways"
- [readme] Parallel python (version 1.6.4, [http://www.parallelpython.com/]: "Non-essential

* Parallel python (version 1.6.4, [http://www.parallelpython.com/]"
- [abstract] compound reference databases (e.g. KEGG, HMDB and LIPID MAPS): "compound reference databases (e.g. KEGG, HMDB and LIPID MAPS)"
