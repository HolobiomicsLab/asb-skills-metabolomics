---
name: compound-class-assignment-from-molecular-formulas
description: Use when you have a peak table (e.g. Report.csv) from direct-injection
  FT-ICR MS where molecular formulas are already assigned and you need elemental/biochemical
  compound-class labels before chemodiversity or Van Krevelen analysis.
when_to_use_negative:
- The input is raw FT-ICR MS spectra with no molecular formulas assigned — MetaboDirect
  requires formula assignment to be performed beforehand (e.g. via MFAssignR)
- The peak table already carries compound-class or elemental-composition annotations
  and only downstream chemodiversity/statistics are needed
- The data set was already filtered by a prior study, as with the S. fallax leachate
  data set where no peaks were filtered
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3292
tools:
- name: MetaboDirect
  role: Command-line pipeline whose data pre-processing step performs compound-class
    assignment on retained molecular formulas
  repo: https://github.com/Coayala/MetaboDirect
- name: Python
  role: Implementation language used to load the Report.csv and metadata.csv tables
    and assign classes per formula
- name: pandas
  role: Used to load and manipulate the peak abundance and molecular formula tables
    during class assignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s40168-023-01476-3
    title: 'MetaboDirect: an analytical pipeline for the processing of FT-ICR MS-based
      metabolomic data'
  source_content_hash: sha256:b5246615e0bc812a863431b0e7ad52f225cf69b9582935ec4206bcebd89c0d19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Coayala/MetaboDirect
  tools:
  - MetaboDirect
  - NumPy
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# compound-class-assignment-from-molecular-formulas

## Summary

Assigns compound classes (elemental classes such as CHO/CHOS/CHON and ClassyFire-style categories) to FT-ICR MS peaks based on their pre-assigned molecular formulas, as implemented in the MetaboDirect data pre-processing step.

## When to use

Use when you have a peak table (e.g. Report.csv) from direct-injection FT-ICR MS where molecular formulas are already assigned and you need elemental/biochemical compound-class labels before chemodiversity or Van Krevelen analysis.

## When NOT to use

- The input is raw FT-ICR MS spectra with no molecular formulas assigned — MetaboDirect requires formula assignment to be performed beforehand (e.g. via MFAssignR)
- The peak table already carries compound-class or elemental-composition annotations and only downstream chemodiversity/statistics are needed
- The data set was already filtered by a prior study, as with the S. fallax leachate data set where no peaks were filtered

## Inputs

- Peak abundance and assigned molecular formula table (e.g. Report.csv) produced after initial processing of raw FT-ICR MS spectra
- Sample metadata table (metadata.csv)
- User-defined m/z range and minimum sample-presence threshold

## Outputs

- .csv tables of filtered peaks annotated with compound classes and thermodynamic indices (NOSC, GFE, AI_mod, DBE)
- Normalized and unnormalized intensity tables
- Elemental and molecular class composition plots

## How to apply

Run the MetaboDirect data pre-processing step on the peak abundance plus assigned-molecular-formula table (e.g. Report.csv) and metadata.csv loaded with pandas. First filter peaks by user-specified m/z range, remove masses whose formulas contain isotopic 13C, keep only formula assignments with error ≤ 0.5 ppm, and retain peaks present in at least the user-defined minimum number of samples. Assign compound classes to the retained formulas from their elemental composition (CHO, CHOS, CHON-style elemental classes and ClassyFire-style categories), alongside computation of thermodynamic indices (NOSC, GFE, AI_mod, DBE) per peak. Export the annotated filtered peak table as .csv files in the pipeline output directory; the class annotations then feed elemental and molecular class composition plots and Van Krevelen diagrams.

## Related tools

- **MetaboDirect** (Command-line pipeline whose data pre-processing step performs compound-class assignment on retained molecular formulas) — https://github.com/Coayala/MetaboDirect
- **Python** (Implementation language used to load the Report.csv and metadata.csv tables and assign classes per formula)
- **pandas** (Used to load and manipulate the peak abundance and molecular formula tables during class assignment)

## Examples

```
pip install metabodirect && metabodirect -h
```

## Evaluation signals

- Isotopic-carbon-containing masses are removed — in the bacterium-phage example ~200 masses were filtered out because their formulas contained 13C
- Every retained peak has a formula assignment error ≤ 0.5 ppm before class assignment
- The filtered peak table with class annotations is exported as .csv files that are readable in the pipeline output directory
- Elemental and molecular class composition plots and Van Krevelen diagrams render from the annotated table without missing-class errors
- No retained formula is empty/unparsable, so chemodiversity and transformation-network steps downstream can consume the class column

## Limitations

- MetaboDirect does not assign molecular formulas itself — formula assignment (e.g. MFAssignR) must be performed before this skill; the pipeline only works on already-assigned formulas
- Compound classes are inferred from formulas, so isomeric compounds are indistinguishable — direct injection cannot separate chemical isomers
- Class coverage is partial: formulas outside the recognized elemental classes (e.g. heavy heteroatom combinations) may remain unclassified

## Evidence

- [intro] detected peaks are filtered by their m/z values (based on the user's input), isotopic presence (13C peaks), error in formula assignment (0.5 ppm): [evidence span withheld — anchor status: not_found]
- [abstract] visualization (e.g., Van Krevelen diagrams, elemental and molecular class composition plots): "visualization (e.g., Van Krevelen diagrams, elemental and molecular class composition plots)"
- [intro] The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis,: "The MetaboDirect pipeline consists of 6 major steps/categories (Fig. 1): (i) data pre-processing, (ii) data diagnostics, (iii) data exploration, (iv) chemodiversity analysis,"
- [abstract] presentation of direct injection high-resolution FT-ICR MS data sets after molecular formula assignment has been performed: "presentation of direct injection high-resolution FT-ICR MS data sets after molecular formula assignment has been performed"
- [readme] MetaboDirect can be installed directly from PyPi: "MetaboDirect can be installed directly from PyPi"
