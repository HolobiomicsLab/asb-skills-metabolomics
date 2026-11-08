---
name: nmr-jres-tissue-distribution-quantification
description: Use when quantifying how metabolites derived from a 13C-labeled amino
  acid diet are distributed across multiple tissues from integrated 13C-JRES spectra,
  to assign each compound an enrichment pattern (liver-enriched, liver-depleted, co-enriched).
when_to_use_negative:
- The spectra are 1D 1H NMR only, without the INADEQUATE-to-JRES integration that
  PyINETA performs — this skill assumes compound identities are already anchored by
  representative JRES peaks.
- The goal is spectral peak picking or compound annotation from raw INADEQUATE data
  rather than downstream tissue-distribution classification of already-annotated compounds.
- The dataset does not contain multiple tissues (e.g., a single tissue time course),
  since the classification logic depends on comparing enrichment across liver, spleen,
  pancreas, muscle, and lung.
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: PyINETA
  role: Used to integrate INADEQUATE peak networks with complementary 13C-JRES data,
    track parameters/outputs, and produce the annotated compound set whose representative
    JRES peaks are then classified across tissues.
  repo: https://github.com/rhilt13/PyINETA
- name: NMRbox
  role: Platform where PyINETA is installed and publicly available for running the
    INADEQUATE-JRES pipeline that feeds the tissue-distribution quantification.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:ed71194d20984f829817f02dc2989792958a826db6e43be615b78ee1ba69cff6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhilt13/PyINETA
  tools:
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# nmr-jres-tissue-distribution-quantification

## Summary

Classify and tabulate tissue-specific enrichment of 13C-labeled metabolites detected by 13C-JRES across five mouse tissues (liver, spleen, pancreas, muscle, lung), yielding discrete enrichment categories (Type-A liver-enriched, Type-B liver-depleted, Type-C co-enriched) grounded in representative JRES peak intensities.

## When to use

Use when quantifying how metabolites derived from a 13C-labeled amino acid diet are distributed across multiple tissues from integrated 13C-JRES spectra, to assign each compound an enrichment pattern (liver-enriched, liver-depleted, co-enriched).

## When NOT to use

- The spectra are 1D 1H NMR only, without the INADEQUATE-to-JRES integration that PyINETA performs — this skill assumes compound identities are already anchored by representative JRES peaks.
- The goal is spectral peak picking or compound annotation from raw INADEQUATE data rather than downstream tissue-distribution classification of already-annotated compounds.
- The dataset does not contain multiple tissues (e.g., a single tissue time course), since the classification logic depends on comparing enrichment across liver, spleen, pancreas, muscle, and lung.

## Inputs

- Integrated 13C-JRES spectra per tissue (liver, spleen, pancreas, muscle, lung) from 13C-labeled amino acid fed mice
- Representative JRES peak intensities for the 19 compounds (produced by PyINETA's INADEQUATE peak-network matching step)
- Simulated INADEQUATE database of metabolites used for compound annotation

## Outputs

- Structured table (CSV/TSV) of the 19 compounds with tissue-by-tissue enrichment assignments
- Enrichment category per compound (Type-A liver-enriched, Type-B liver-depleted, Type-C co-enriched)
- Category counts matching Figure 3 (13 Type-A, 2 Type-B, 4 Type-C)

## How to apply

Load the tissue JRES compound data for the labeled-fed mice covering the 19 compounds under investigation. For each compound, extract a representative JRES peak (the article's pipeline matches INADEQUATE networks to a simulated database, then integrates complementary 13C-JRES information) and quantify its relative intensity across liver, spleen, pancreas, muscle, and lung. Compare peak heights/areas across tissues to assign one of three enrichment categories: Type-A (highest in liver), Type-B (depleted in liver but enriched in pancreas and/or spleen, e.g., O-phosphorylethanolamine, hypotaurine), or Type-C (co-enriched in liver and other tissues, e.g., valine, threonine, isoleucine, taurine). Tabulate the 19 compounds with their tissue-by-tissue assignments in a structured table (CSV/TSV) and verify the resulting category counts (13 Type-A, 2 Type-B, 4 Type-C) against the paper's Figure 3. PyINETA tracks all pipeline parameters and outputs, so preserve these logs alongside the quantification table to ensure annotation transparency.

## Related tools

- **PyINETA** (Used to integrate INADEQUATE peak networks with complementary 13C-JRES data, track parameters/outputs, and produce the annotated compound set whose representative JRES peaks are then classified across tissues.) — https://github.com/rhilt13/PyINETA
- **NMRbox** (Platform where PyINETA is installed and publicly available for running the INADEQUATE-JRES pipeline that feeds the tissue-distribution quantification.)

## Examples

```
python ../../run_pyineta.py -c config.ini -o example1_output
```

## Evaluation signals

- The resulting table contains exactly 19 compounds (13 Type-A, 2 Type-B, 4 Type-C) matching the paper's Figure 3 counts.
- Each compound must show enrichment in at least one tissue — no compound should be unclassified or left with all tissues at equal relative intensity.
- O-phosphorylethanolamine and hypotaurine must land in Type-B (liver-depleted) while valine, threonine, isoleucine, and taurine must land in Type-C (co-enriched).
- All pipeline parameters and outputs used to generate the representative JRES peaks are preserved (PyINETA keeps track of these automatically), enabling re-annotation transparency.
- Each category assignment is reproducible when re-run: a second pass over the same tissue JRES data with the same representative peaks must yield identical category labels.

## Limitations

- Representative JRES peaks depend on the upstream INADEQUATE network-matching being correct; errors in peak picking or network filtering will propagate into misclassified tissue enrichment.
- The article does not state numerical thresholds for what counts as 'enriched' or 'depleted'; the classification is relative across the five tissues rather than based on an absolute cutoff.
- PyINETA provides fragment information for unknown compounds but does not guarantee full structural annotation of all 19 metabolites, so some compounds may rely on representative peaks without confirmed identity.

## Evidence

- [abstract] tissue distribution category classification: "The distribution of those compounds was tissue-specific, showing enrichment of particular metabolites in liver, spleen, pancreas, muscle, or lung."
- [intro] representative JRES peaks: "integrates information of INADEQUATE and a complementary NMR experiment 13 C J -resolved experiment ( 13 C-JRES)"
- [discussion] category verification: "making the best use of 13C NMR and improving the transparency and reproducibility of compound annotation"
- [readme] pipeline provenance: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify"
