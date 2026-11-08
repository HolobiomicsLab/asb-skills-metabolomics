---
name: fragment-ion-search-parameter-configuration
description: Use when dereplicating experimental CID/HCD tandem MS spectra of NRPs
  against a CycloBranch sequence database and you must set error tolerances, intensity
  thresholds, and ion-type filters for a fair tool comparison.
when_to_use_negative:
- You are running Dereplicator or Dereplicator+ on GNPS, which require 0.005 Da tolerances
  rather than the 5 ppm values used here for CycloBranch/NRPro.
- The spectra are of peptide-like compounds fully covered by CycloBranch's built-in
  default building-block database and no custom MassSpecBlocks-exported database is
  needed.
- You need NRPro itself or custom building-block databases inside the search engine
  — the article notes users cannot create custom databases of building blocks and
  NRP sequences in NRPro.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3520
tools:
- name: CycloBranch
  role: Executes the dereplication comparator against the MassSpecBlocks-exported
    sequence database under the configured tolerances and ion filters.
  repo: https://ms.biomed.cas.cz/cyclobranch/docs/html/
- name: MassSpecBlocks
  role: Produces the plain text sequence/building-block database that the configured
    CycloBranch comparator searches.
  repo: https://github.com/privrja/MassSpecBlocks
- name: NRPro
  role: Reference tool whose 5 ppm tolerance and b/a-ion + one neutral loss configuration
    CycloBranch is unified with.
- name: Dereplicator+
  role: Alternative comparator requiring different (0.005 Da) tolerance settings when
    benchmarking across tools.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-021-00530-2
    title: 'MassSpecBlocks: a web-based tool to create building blocks and sequences
      of nonribosomal peptides and polyketides for tandem mass spectra analysis'
  source_content_hash: sha256:80c10ef33bf3258a6ec4800ae563b332f5d93eab5e3581d12c89be66ed1ee4ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CycloBranch
  - Dereplicator+
  - MassSpecBlocks
  - NRPro
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# fragment-ion-search-parameter-configuration

## Summary

Configures CycloBranch comparator searches against MassSpecBlocks-exported sequence databases by unifying precursor/fragment m/z tolerances, peak-intensity thresholds, and ion-series/neutral-loss settings with NRPro's defaults so dereplication accuracy is comparable across tools.

## When to use

Use when dereplicating experimental CID/HCD tandem MS spectra of NRPs against a CycloBranch sequence database and you must set error tolerances, intensity thresholds, and ion-type filters for a fair tool comparison.

## When NOT to use

- You are running Dereplicator or Dereplicator+ on GNPS, which require 0.005 Da tolerances rather than the 5 ppm values used here for CycloBranch/NRPro.
- The spectra are of peptide-like compounds fully covered by CycloBranch's built-in default building-block database and no custom MassSpecBlocks-exported database is needed.
- You need NRPro itself or custom building-block databases inside the search engine — the article notes users cannot create custom databases of building blocks and NRP sequences in NRPro.

## Inputs

- Experimental tandem mass spectra acquired under CID and HCD (e.g., 44 compounds × 2 modes = 88 spectra)
- MassSpecBlocks-exported sequence database in CycloBranch plain text format
- Comparator search configuration: ppm tolerances, intensity threshold, ion types, neutral losses, precursor mass filter on/off

## Outputs

- Top-scoring compound hit per spectrum
- Per-spectrum matched-peak statistics (e.g., weighted ratio of matched peaks, 96.4% for desferrioxamine B top hit)
- Per-fragmentation-mode dereplication accuracy values for export (e.g., comparator_accuracy_cid_hcd.csv)
- CycloBranch output report (enabled vs disabled precursor mass filter variants)

## How to apply

Before running the CycloBranch comparator, set the precursor and fragment ion m/z error tolerances to 5 ppm and the minimum threshold of relative peak intensity to zero. Restrict the search to b-ions and a-ions only, allowing at most one neutral loss of H2O or NH3 — this unifies the CycloBranch configuration with NRPro, which uses identical 5 ppm tolerances. Choose ion-series and neutral-loss settings deliberately rather than defaults, because Dereplicator/Dereplicator+ require different tolerance values (0.005 Da) and would otherwise not be comparable. Decide whether to enable the precursor mass filter depending on whether you want the top-scoring hit constrained by precursor mass; the article shows both enabled and disabled output reports for desferrioxamine B. Run the comparator separately per fragmentation mode (CID vs HCD) and record the top-scoring compound for each spectrum to compute per-mode accuracy.

## Related tools

- **CycloBranch** (Executes the dereplication comparator against the MassSpecBlocks-exported sequence database under the configured tolerances and ion filters.) — https://ms.biomed.cas.cz/cyclobranch/docs/html/
- **MassSpecBlocks** (Produces the plain text sequence/building-block database that the configured CycloBranch comparator searches.) — https://github.com/privrja/MassSpecBlocks
- **NRPro** (Reference tool whose 5 ppm tolerance and b/a-ion + one neutral loss configuration CycloBranch is unified with.)
- **Dereplicator+** (Alternative comparator requiring different (0.005 Da) tolerance settings when benchmarking across tools.)

## Examples

```
"
```

## Evaluation signals

- Computed CID and HCD comparator accuracies match the accuracies reported in the paper's evaluation.
- Precursor and fragment ion m/z error tolerances are recorded as 5 ppm (not 0.005 Da) in the CycloBranch configuration.
- Search yields b-ions and a-ions only, with at most one neutral loss of H2O or NH3, mirroring the NRPro-unified configuration.
- Desferrioxamine B is reported as the single hit when the precursor mass filter is enabled, with a weighted matched-peak ratio near 96.4%.
- Minimum relative peak intensity threshold is set to zero before running the comparator.

## Limitations

- When multiple building-block sequences share the same precursor mass, enabling the precursor mass filter alone may not resolve the ambiguity — the article shows both an enabled and a disabled filter report for desferrioxamine B.
- Limiting the search to b-ions and a-ions with only one H2O or NH3 neutral loss (chosen to unify with NRPro) excludes other ion series that might improve matching for some compounds.
- PK siderophores such as desferrioxamines contain uncommon building blocks that reverse peptide-bond orientation, further complicating mass spectra interpretation.
- Tolerance values are tool-specific: 5 ppm for CycloBranch/NRPro but 0.005 Da for Dereplicator and Dereplicator+, so the same config cannot be reused verbatim across engines.

## Evidence

- [results] Precursor and fragment ion m/z tolerances unified at 5 ppm: "The precursor and fragment ion m/z error tolerances were set up to 5ppm for CycloBranch and NRPro. The values used in Dereplicator and Dereplicator+ were 0.005 Da."
- [results] Intensity threshold and ion-series restrictions chosen to unify with NRPro: "we set up the minimum threshold of relative peak intensity to zero."
- [results] Ion types and neutral losses restricted for cross-tool comparability: "we were looking only for b-ions, a-ions and allowed only one neutral loss of H2O or NH3."
- [results] Precursor mass filter toggled between output reports: "In Additional file 1: Fig. S13, we show an output report if a precursor mass filter was enabled."
- [readme] MassSpecBlocks exports plain text databases consumed by CycloBranch: "The application provides an export format for open source program [CycloBranch](https://ms.biomed.cas.cz/cyclobranch/docs/html/) from Jiří Novák"
