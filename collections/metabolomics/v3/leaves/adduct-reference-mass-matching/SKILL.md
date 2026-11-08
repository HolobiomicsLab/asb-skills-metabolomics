---
name: adduct-reference-mass-matching
description: Use when you have a binned mass-difference histogram from MALDI-MSI peak
  pairs and want to count occurrences of known adducts (Na+, K+, 13C, DHB or CHCA
  matrix adducts) within instrument mass tolerance.
when_to_use_negative:
- Your goal is to verify spatial co-localization of the candidate parent–adduct pairs
  — mass matching alone does not test whether parent and adduct ion images are correlated;
  follow up with corrPairsMSI.
- Your input is an unbinned list of raw pairwise mass differences; adductMatch operates
  on the histogram, so you must bin first with hist().
- The adduct you care about is not in the reference table and you have not built a
  custom name/formula/mass table — the matching will silently miss it (use topAdducts
  to survey unmatched high-count bins).
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mass2adduct
  role: R package providing adductMatch() to match binned mass differences against
    the built-in adducts/adducts2 reference tables, and topAdducts() to rank mass
    differences by occurrence and flag matches.
  repo: github.com/kbseah/mass2adduct
- name: R
  role: Runtime environment for the mass2adduct functions; package is installed via
    devtools::install_github().
  repo: https://cran.r-project.org/web/packages/devtools/index.html
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.0c04720
    title: Determination of Abundant Metabolite Matrix Adducts Illuminates the Dark
      Metabolome of MALDI-Mass Spectrometry Imaging Datasets
  source_content_hash: sha256:f4c5e1c03bcea281fc10ce2dd3a05be77d6430667f811148da01f3181f319a3f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cardinal
  - MSiReader
  - R
  - SCiLS
  - mass2adduct
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# adduct-reference-mass-matching

## Summary

Annotates candidate parent–adduct ion pairs in MALDI-MSI peak lists by matching binned pairwise mass differences against the mass2adduct built-in reference tables (adducts / adducts2), reporting how many peak pairs fall into the histogram bin closest to each known adduct mass.

## When to use

Use when you have a binned mass-difference histogram from MALDI-MSI peak pairs and want to count occurrences of known adducts (Na+, K+, 13C, DHB or CHCA matrix adducts) within instrument mass tolerance.

## When NOT to use

- Your goal is to verify spatial co-localization of the candidate parent–adduct pairs — mass matching alone does not test whether parent and adduct ion images are correlated; follow up with corrPairsMSI.
- Your input is an unbinned list of raw pairwise mass differences; adductMatch operates on the histogram, so you must bin first with hist().
- The adduct you care about is not in the reference table and you have not built a custom name/formula/mass table — the matching will silently miss it (use topAdducts to survey unmatched high-count bins).

## Inputs

- A binned mass-difference histogram (R histogram object) produced by hist() on a massdiff object from mass2adduct
- Reference adduct table: built-in adducts or adducts2 data.frame, or a custom data.frame with columns name, formula, mass

## Outputs

- Matched-adduct table reporting, per reference adduct, the count of peak pairs in the closest-matching histogram bin and the bin's quantile
- Ranked list of highest-counted mass differences with any matches to known adducts (from topAdducts)

## How to apply

First compute all pairwise mass differences with massdiff() and bin them into a histogram whose bin width reflects the instrument's known mass precision (e.g. 0.01 in the README example), since the raw differences are 'misleadingly precise' because measurement error is not accounted for. Then call adductMatch(d.diff.hist), which 'looks for known adducts by finding the closest-matching bin in the mass difference histogram' and reports, for each reference adduct, the number of peak pairs in that bin and the bin's quantile. The package ships two reference tables, adducts (full) and adducts2 (shorter); users may supply a custom data.frame with columns name, formula, mass instead. Quantiles are usually high because most mass differences have zero to few counts, so interpret counts relative to this baseline. Use topAdducts(d.diff.hist, n=10) as the complementary view — ranking mass differences by occurrence and reporting any adduct matches — to catch histogram peaks that the reference list misses.

## Related tools

- **mass2adduct** (R package providing adductMatch() to match binned mass differences against the built-in adducts/adducts2 reference tables, and topAdducts() to rank mass differences by occurrence and flag matches.) — github.com/kbseah/mass2adduct
- **R** (Runtime environment for the mass2adduct functions; package is installed via devtools::install_github().) — https://cran.r-project.org/web/packages/devtools/index.html

## Examples

```
d.diff.hist <- hist(d.diff, binwidth=0.01); head(adductMatch(d.diff.hist))
```

## Evaluation signals

- The output table lists known adducts from the reference list with a paired-count column; verify the count corresponds to the histogram bin nearest each adduct mass.
- Sanity-check quantiles: they 'will usually be quite high because the majority of mass differences have zero to few counts', so high quantiles alone are not evidence of a real adduct.
- Cross-check with topAdducts(): high-count mass differences should appear there; discrepancies flag reference-list gaps.
- Bin width must be chosen to match instrument mass precision (README uses 0.01); re-binning at a coarser/finer width should reproduce the same top matches within tolerance.
- Matches whose mass difference exceeds the chosen tolerance window (e.g. 1 mDa or ppm proportion for DHB-H2O) should not be counted as adduct hits.

## Limitations

- Quantile values in the output will usually be quite high because most mass differences have zero to few counts, so they are not by themselves indicative of adduct abundance.
- Only adducts present in the reference table (or a user-supplied custom table) can be matched; adductMatch misses histogram peaks without a reference match — use topAdducts to find those.
- The matching is tolerant only to the histogram bin width; 'mass differences are misleadingly precise, because measurement error and uncertainty are not taken into account', so bin width must reflect instrument mass accuracy.
- Mass matching alone does not establish that a candidate pair is a true adduct; spatial correlation testing (corrPairsMSI, Bonferroni-corrected p=0.05) is a separate downstream step.

## Evidence

- [readme] adductMatch looks for known adducts by finding the closest-matching bin in the mass difference histogram: "The following function looks for known adducts by finding the closest-matching bin in the mass difference histogram produced above."
- [methods] reference tables adducts and adducts2: "There are two built-in data sets `adducts` and `adducts2` (shorter), which list biologically-relevant chemical species that might occur in biological samples."
- [readme] bin width depends on instrument mass precision: [evidence span withheld — anchor status: external_doc]
- [methods] topAdducts ranks mass differences by their occurrences: "`topAdducts` ranks mass differences by their occurrences, and reports them in descending order, as well as matches to known adducts, if any"
- [readme] quantiles usually high: "Note that quantile values will usually be quite high because the majority of mass differences have zero to few counts."
