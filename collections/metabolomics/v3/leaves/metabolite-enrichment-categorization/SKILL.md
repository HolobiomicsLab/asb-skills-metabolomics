---
name: metabolite-enrichment-categorization
description: Use when INADEQUATE-annotated metabolites from 13C-labeled amino acid
  feeding need to be assigned to tissue-enrichment categories across multiple organs
  (e.g. liver, spleen, pancreas, muscle, lung) to reproduce PyINETA tissue-distribution
  output.
when_to_use_negative:
- When the goal is de novo compound annotation from raw INADEQUATE spectra — use the
  PyINETA peak-picking/clustering/matching pipeline instead, not the enrichment categorization
  step.
- When tissue samples are from unlabeled (natural abundance) feeding experiments,
  since tissue-specific enrichment of labeled metabolites is the basis of this categorization.
- When only a single tissue (e.g. liver-only spectra) is available, since Type-B and
  Type-C calls require relative abundance across multiple tissues.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PyINETA
  role: Provides the compound-level annotated 13C-JRES peak intensities across tissues
    that serve as the input for tissue-enrichment categorization; its tissue-distribution
    analysis is the reference output being reproduced
  repo: https://github.com/edisonomics/PyINETA
- name: Python
  role: Used to load and tabulate tissue-by-tissue peak intensities and assign Type-A/B/C
    enrichment categories into a CSV/TSV summary
  repo: https://github.com/edisonomics/PyINETA
- name: NMRbox
  role: Publicly available platform where PyINETA is installed for running the INADEQUATE–JRES
    integration that generates the input enrichment data
  repo: https://www.nmrbox.org
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:fd531ded84a65763073e0b609757727f220a822a32d86e231583b30634ac78cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolite-enrichment-categorization

## Summary

Classify metabolites traced from a 13C-labeled diet into tissue-distribution categories (liver-enriched, liver-depleted, co-enriched) by quantifying their representative 13C-JRES peak intensities across mouse tissues, reproducing the PyINETA tissue-distribution analysis.

## When to use

Use when INADEQUATE-annotated metabolites from 13C-labeled amino acid feeding need to be assigned to tissue-enrichment categories across multiple organs (e.g. liver, spleen, pancreas, muscle, lung) to reproduce PyINETA tissue-distribution output.

## When NOT to use

- When the goal is de novo compound annotation from raw INADEQUATE spectra — use the PyINETA peak-picking/clustering/matching pipeline instead, not the enrichment categorization step.
- When tissue samples are from unlabeled (natural abundance) feeding experiments, since tissue-specific enrichment of labeled metabolites is the basis of this categorization.
- When only a single tissue (e.g. liver-only spectra) is available, since Type-B and Type-C calls require relative abundance across multiple tissues.

## Inputs

- 13C-JRES integrated peak intensity table per compound per tissue (liver, spleen, pancreas, muscle, lung) from 13C-labeled amino acid fed mice
- PyINETA compound annotation output (compound names / representative peak assignments) for the 19 traced metabolites

## Outputs

- Structured CSV/TSV table of compounds with tissue-by-tissue enrichment assignments (Type-A / Type-B / Type-C)
- Category counts (e.g. 13 Type-A liver-enriched, 2 Type-B liver-depleted, 4 Type-C co-enriched) for comparison with the publication's tissue-distribution figure

## How to apply

Load the 13C-JRES compound-level peak data covering the annotated compounds (e.g. 19 compounds from the 13C-labeled amino acid fed mice) across each tissue sample (liver, spleen, pancreas, muscle, lung). For each compound, quantify its representative peak intensity across tissues using the values PyINETA reported from its INADEQUATE–JRES integration. Assign each compound to a distribution category based on which tissues show highest relative abundance: Type-A (enriched in liver), Type-B (depleted in liver but enriched in pancreas and/or spleen), Type-C (co-enriched in liver and other tissues). Tabulate compound-to-tissue assignments in a structured CSV/TSV so counts per category can be compared against the published tissue-distribution figure (e.g. 13 Type-A, 2 Type-B, 4 Type-C). The rationale is that PyINETA's tissue-distribution analysis relies on tissue-specific enrichment patterns of labeled compounds rather than absolute concentrations.

## Related tools

- **PyINETA** (Provides the compound-level annotated 13C-JRES peak intensities across tissues that serve as the input for tissue-enrichment categorization; its tissue-distribution analysis is the reference output being reproduced) — https://github.com/edisonomics/PyINETA
- **Python** (Used to load and tabulate tissue-by-tissue peak intensities and assign Type-A/B/C enrichment categories into a CSV/TSV summary) — https://github.com/edisonomics/PyINETA
- **NMRbox** (Publicly available platform where PyINETA is installed for running the INADEQUATE–JRES integration that generates the input enrichment data) — https://www.nmrbox.org

## Examples

```
python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- Compound counts per category should match the published tissue-distribution figure (13 Type-A liver-enriched, 2 Type-B liver-depleted, 4 Type-C co-enriched, totaling 19 compounds).
- Every compound must be assigned to exactly one of the three categories (Type-A, Type-B, or Type-C) with no unclassified rows.
- Type-B compounds (e.g. O-phosphorylethanolamine, hypotaurine) should show higher intensity in pancreas and/or spleen than in liver.
- Type-C compounds (e.g. valine, threonine, isoleucine, taurine) should show high intensity in liver and at least one other tissue.
- The CSV/TSV output must contain one row per compound with a tissue-by-tissue enrichment assignment that sums across tissues to a single category label.

## Limitations

- Categorization is relative (which tissues are enriched) and depends on PyINETA's successful INADEQUATE–JRES integration; mis-annotated compounds would propagate incorrect tissue assignments.
- Only compounds that PyINETA annotated from the 13C-labeled feeding experiment can be categorized — unknown fragments or unannotated compounds fall outside this classification.
- The analysis assumes the labeled amino acids were transferred and transformed to other metabolites across tissues; if labeling efficiency or tissue extraction differs, category assignments may shift.

## Evidence

- [results] The analysis of 19 compounds via representative JRES peaks revealed three distribution categories: 13 compounds enriched in liver (Type-A), 2 metabolites depleted in liver but enriched in pancreas and/or spleen (Type-B, including O-phosphorylethanolamine and hypotaurine), and 4 compounds enriched in both liver and other tissues (Type-C, including valine, threonine, isoleucine, and taurine).: "The distribution of those compounds was tissue-specific, showing enrichment of particular metabolites in liver, spleen, pancreas, muscle, or lung."
- [abstract] For each compound, quantify its representative peak intensity or presence across each of the five tissues (liver, spleen, pancreas, muscle, lung) from the integrated 13C-JRES data.: "Our evaluation of PyINETA using a model mouse study showed that our pipeline successfully integrated INADEQUATE and 13 C-JRES."
- [abstract] 13C-labeled amino acids that were fed to mice were transferred to different tissues, and, also, they were transformed to other metabolites.: "The results showed that 13 C-labeled amino acids that were fed to mice were transferred to different tissues, and, also, they were transformed to other metabolites."
- [methods] PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics.: "Those steps are carried out automatically, and PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
- [readme] Load the tissue JRES compound data for the three 13C-labeled-amino-acid fed mice covering the 19 compounds.: "pyINETA can perform basic tasks such as reading and referencing (using basic shifting) the INADEQUATE spectra and peak picking."
