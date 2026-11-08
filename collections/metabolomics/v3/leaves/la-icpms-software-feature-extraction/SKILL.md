---
name: la-icpms-software-feature-extraction
description: Use when comparing LA-ICP-MS imaging software packages on concrete capabilities
  (e.g. which vendors' raw batch formats they natively import, whether segmentation
  or colocalization is built in) or when auditing a new tool's feature set against
  published alternatives.
when_to_use_negative:
- The goal is to actually run LA-ICP-MS image processing (import, calibrate, segment)
  rather than to audit or compare software feature sets — use the target tool directly
  instead
- The source article contains no explicit capability statements for a given software
  package and no comparison table to cross-reference — inventing cells would produce
  a misleading matrix
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_0092
tools:
- name: Pew2
  role: The software whose capabilities are enumerated and compared in the feature
    matrix; capabilities are read from the article's Results text
  repo: https://github.com/djdt/pewpew
- name: pandas
  role: Used to construct the rectangular feature-by-software matrix and to write/validate
    the TSV output
- name: pewlib
  role: Python library referenced in the article and README that implements import/export
    of LA-ICP-MS data; its supported formats supplement the capability list
  repo: https://github.com/djdt/pewlib
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:6a0cf6ff03cf2eb65d5b05125629ddcfb648cd0a61d13f77730af47bcbb03065
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - Pew2
  - pandas
  - pewlib
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# la-icpms-software-feature-extraction

## Summary

A method for systematically extracting and tabulating LA-ICP-MS imaging software capabilities — vendor import formats, peak detection, calibration, drift correction, filtering, segmentation, and colocalization — from a paper's text and comparison tables, then building a rectangular feature-by-software matrix. It enables evidence-based software selection and gap analysis across the LA-ICP-MS data-reduction landscape.

## When to use

Use when comparing LA-ICP-MS imaging software packages on concrete capabilities (e.g. which vendors' raw batch formats they natively import, whether segmentation or colocalization is built in) or when auditing a new tool's feature set against published alternatives.

## When NOT to use

- The goal is to actually run LA-ICP-MS image processing (import, calibrate, segment) rather than to audit or compare software feature sets — use the target tool directly instead
- The source article contains no explicit capability statements for a given software package and no comparison table to cross-reference — inventing cells would produce a misleading matrix

## Inputs

- Scientific article text describing LA-ICP-MS software capabilities (e.g. the Results section of the Pew2 paper)
- Software-comparison tables from the article (e.g. Table 1 listing competing LA-ICP-MS packages)
- Repository README or documentation enumerating supported import formats (e.g. github.com/djdt/pewpew README)

## Outputs

- Feature-by-software comparison matrix (rows = capabilities, columns = software packages)
- TSV file with header row and capability column, parseable by pandas into a rectangular table

## How to apply

Read the article's Results section and enumerate every software capability explicitly reported — for Pew2 this includes drag-and-drop import of Agilent MassHunter G7200B+ batches, Perkin Elmer Elan XL directories, and Thermo Fischer Qtegra exports, plus text image and file-per-line importers; peak detection by simple/windowed thresholding and continuous wavelet transforms; calibration with r2 up to 0.9959; polynomial drift renormalization; local mean/median spike filtering; segmentation via three-level k-means, mean thresholding, and Otsu's method; and RGB overlay with Pearson/Li-ICQ colocalization. Cross-reference the paper's software-comparison table (e.g. Table 1) to enumerate competing packages and their listed features, marking cells only where the article explicitly states a capability and leaving cells blank where it does not. Build the matrix with pandas (rows = capabilities, columns = software packages including the paper's tool), then export as a TSV with a header row and a capability column. Validate that the output parses to a rectangular table and contains every extracted capability plus all software columns.

## Related tools

- **Pew2** (The software whose capabilities are enumerated and compared in the feature matrix; capabilities are read from the article's Results text) — https://github.com/djdt/pewpew
- **pandas** (Used to construct the rectangular feature-by-software matrix and to write/validate the TSV output)
- **pewlib** (Python library referenced in the article and README that implements import/export of LA-ICP-MS data; its supported formats supplement the capability list) — https://github.com/djdt/pewlib

## Examples

```
import pandas as pd; df = pd.read_csv('features.tsv', sep='\t'); assert df.shape == df.dropna(axis=1, how='all').shape and all(df.columns[1:])
```

## Evaluation signals

- Output TSV parses with pandas read_csv(sep='\t') into a rectangular DataFrame with no ragged rows
- Every capability explicitly stated in the article's Results text appears as a row in the matrix
- The matrix includes a column for every software package enumerated in the paper's comparison table (e.g. Table 1) plus the paper's own tool
- Cells are blank (not filled with 'unknown' or guessed values) where the article does not state a capability for a given package
- Header row and capability column are present in the exported TSV

## Limitations

- Feature cells can only be populated where the article explicitly states a capability; unreported features must be left blank to avoid overstating a package's functionality
- The README's supported-format list may evolve independently of the publication, so README-derived formats (e.g. Nu Instruments Vitesse, ImzML, Iolite laser-log sync) should be noted as repository-level evidence distinct from paper-stated capabilities
- Spot-wise import in Pew2 relies on a single 'marker' element being present across the entire sample with similar peak positions per element, which constrains which datasets the software handles without user intervention

## Evidence

- [results] Drag-and-drop importation of raw Agilent batches (Mass-Hunter G7200B or later), Perkin Elmer Elan 'XL', and Thermo Fischer Qtegra exports, as well as both text image and file-per-line exports: [evidence span withheld — anchor status: not_found]
