---
name: adduct-annotation-against-reference-table
description: Use when you have a mass-difference histogram (or pairwise mass differences)
  from MS peaks and need to label which differences correspond to known metabolite–matrix
  or salt adducts before spatial correlation or peak filtering.
when_to_use_negative:
- When you need confident ion identities rather than putative annotations — adduct
  matches are closest-bin assignments and require subsequent spatial correlation (corrPairsMSI)
  to support parent–adduct relationships
- When mass differences cannot be binned reliably because instrument mass precision
  is unknown or too low, making closest-bin matching meaningless
- When your analytes/adducts are absent from both `adducts2` and any custom table
  you can supply — use topAdducts-driven discovery instead
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: mass2adduct
  role: Provides adductMatch, topAdducts, diffGetPeaks, and the built-in adducts/adducts2
    reference tables used to annotate mass differences
  repo: github.com/kbseah/mass2adduct
- name: R
  role: Execution environment; mass2adduct is installed from GitHub via devtools::install_github
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:6ffea1513d8ae562c6e8b2629c1ae7528101d9ae5775c4f0c1f1d8007df9f34d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - mass2adduct
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# adduct-annotation-against-reference-table

## Summary

Annotates candidate molecular adducts from MALDI-MSI mass-difference data by matching histogrammed peak-pair mass differences against a reference table of known biologically relevant adducts (e.g. mass2adduct's built-in `adducts`/`adducts2`), converting raw m/z differences into putative adduct assignments such as DHB matrix adducts.

## When to use

Use when you have a mass-difference histogram (or pairwise mass differences) from MS peaks and need to label which differences correspond to known metabolite–matrix or salt adducts before spatial correlation or peak filtering.

## When NOT to use

- When you need confident ion identities rather than putative annotations — adduct matches are closest-bin assignments and require subsequent spatial correlation (corrPairsMSI) to support parent–adduct relationships
- When mass differences cannot be binned reliably because instrument mass precision is unknown or too low, making closest-bin matching meaningless
- When your analytes/adducts are absent from both `adducts2` and any custom table you can supply — use topAdducts-driven discovery instead

## Inputs

- massdiff object or mass-difference histogram (R histogram object from hist(d.diff))
- Adduct reference table: built-in mass2adduct datasets `adducts` or `adducts2`, or a custom data.frame with columns name, formula, mass
- Optional: msimat object for downstream peak-pair extraction via diffGetPeaks

## Outputs

- Data.frame of adduct matches with counts (number of peak pairs at each mass difference) and quantile values
- Ranked list from topAdducts of highest-count mass differences with any adduct matches
- Subset massdiff object of parent/adduct peak pairs for a specific adduct (e.g. DHB-H2O, mass 136.01600 ± 0.5 mDa)

## How to apply

First bin all pairwise mass differences from a `massdiff` object into a histogram with a bin width matched to your instrument's mass precision (e.g. 0.01 Da), because raw differences are misleadingly precise. Then run `adductMatch()` on the histogram: it finds the closest-matching bin for each species in the reference table (built-in `adducts` or `adducts2` datasets) and reports the count of peak pairs with that difference plus a quantile. Complement it with `topAdducts(n=10)` to surface high-count mass differences that have no reference match, and use `diffGetPeaks(d.diff, mass=136.01600, mDa=1)` to pull the specific parent/adduct peak pairs for a targeted adduct such as DHB-H2O. Custom reference lists are allowed as a data.frame with `name`, `formula`, `mass` columns.

## Related tools

- **mass2adduct** (Provides adductMatch, topAdducts, diffGetPeaks, and the built-in adducts/adducts2 reference tables used to annotate mass differences) — github.com/kbseah/mass2adduct
- **R** (Execution environment; mass2adduct is installed from GitHub via devtools::install_github)

## Examples

```
d.diff.hist <- hist(d.diff); head(adductMatch(d.diff.hist)); d.diff.DHBH2O <- diffGetPeaks(d.diff, mass=136.01600, mDa=1)
```

## Evaluation signals

- adductMatch output is a data.frame where each known adduct has an integer count of peak pairs and a quantile; quantiles are expected to be high since most mass differences have zero to few counts
- Sanity check: plotting the histogram (plot(d.diff.hist)) and zooming with xlim should show peaks near the mass of targeted adducts, e.g. 136.01600 for DHB-H2O
- diffGetPeaks with mass=136.01600 and mDa=1 must return a subset massdiff object containing plausible parent–adduct pairs (window ±0.5 mDa around the adduct mass)
- Custom reference tables must pass the schema check: exactly three columns named name, formula, mass
- Biological plausibility from the paper: adduct-forming metabolites with DHB in positive mode contained at least one amine group

## Limitations

- Quantile values from adductMatch are usually high because the majority of mass differences have zero to few counts, so quantiles alone do not indicate significance
- Mass differences are misleadingly precise; the histogram bin width must be chosen based on the instrument's known mass precision, and matching is to the closest bin only
- adductMatch only finds known adducts; unmatched histogram peaks require the complementary topAdducts function

## Evidence

- [methods] Built-in adduct reference tables: "There are two built-in data sets `adducts` and `adducts2` (shorter), which list biologically-relevant chemical species that might occur in biological samples."
- [readme] adductMatch mechanism: "The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above."
- [readme] Custom reference table format: "Users can supply their own custom sets of adducts as long as they are in the same format (as a data.frame with three columns: `name`, `formula`, `mass`)"
- [readme] Bin width rationale: [evidence span withheld — anchor status: external_doc]
- [readme] Targeted adduct extraction example: [evidence span withheld — anchor status: external_doc]
