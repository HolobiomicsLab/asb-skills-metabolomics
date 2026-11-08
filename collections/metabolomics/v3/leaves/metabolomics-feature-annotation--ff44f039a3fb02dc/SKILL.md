---
name: metabolomics-feature-annotation
description: Use when rerunning khipu annotation on a preprocessed feature table (e.g.
  from asari) and the default isotope/adduct patterns leave many features unexplained,
  so experiment-specific adducts must be supplied to extend coverage.
when_to_use_negative:
- Input is raw LC-MS data rather than a preprocessed feature table — khipu expects
  feature ID, m/z, rtime and intensity columns from asari or another preprocessing
  tool.
- The goal is complete, definitive metabolite annotation — khipu is a preannotation
  step, and complete annotation still requires contaminants lists, authentic libraries,
  and tandem MS data.
- Default search patterns already explain the data adequately and no experiment-specific
  adducts or isotope-tracing patterns are needed.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: khipu
  role: Python 3 package performing the mass-difference search, mass-distance-network
    to DAG conversion, adduct-tree alignment, and neutral-mass regression; search
    patterns are user-overwritable to extend adduct coverage
  repo: https://github.com/shuzhao-li/khipu
- name: asari 1.9.2
  role: Preprocessing tool that generates the feature tables (and the ESI- SNR>100-filtered
    demo) that khipu annotates
  repo: https://github.com/shuzhao-li/asari
- name: Jupyter
  role: Demo notebooks under notebooks/ demonstrate how to supply custom isotope and
    adduct patterns when rerunning annotation
- name: mass2chem
  role: Provides the search functions used by khipu for pattern matching
  repo: https://github.com/shuzhao-li-lab/mass2chem
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:4865368ac2384bb25be80157617716efc95bcce1fcfa25480c8446f980256c44
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shuzhao-li/khipu
  tools:
  - Jupyter
  - Python 3
  - asari
  - asari 1.9.2
  - khipu
  - mass2chem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolomics-feature-annotation

## Summary

Annotates LC-MS features into empirical compounds using khipu's generalized tree structure, including rerunning annotation with user-supplied (custom) isotope/adduct search patterns to extend adduct coverage beyond the defaults. Demonstrated on yeast metabolomics data (MassIVE MSV000087434) processed with asari 1.9.2, where extended adducts raised explained ESI+ features from 6,310 to 8,049 and ESI- from 2,601 to 2,912.

## When to use

Use when rerunning khipu annotation on a preprocessed feature table (e.g. from asari) and the default isotope/adduct patterns leave many features unexplained, so experiment-specific adducts must be supplied to extend coverage.

## When NOT to use

- Input is raw LC-MS data rather than a preprocessed feature table — khipu expects feature ID, m/z, rtime and intensity columns from asari or another preprocessing tool.
- The goal is complete, definitive metabolite annotation — khipu is a preannotation step, and complete annotation still requires contaminants lists, authentic libraries, and tandem MS data.
- Default search patterns already explain the data adequately and no experiment-specific adducts or isotope-tracing patterns are needed.

## Inputs

- Tab-delimited feature table from asari 1.9.2 (columns: feature ID, m/z, rtime, intensities; start/end intensity columns configurable)
- Yeast metabolomics feature tables from MassIVE MSV000087434 (ESI+ and SNR>100-filtered ESI-)
- User-supplied isotope/adduct search patterns (e.g. additional adducts from Chen et al.)
- Built-in khipu grid patterns: 1.0034 Da (13C/12C), 22.9893 Da (Na+), M+H, M+NH4, M+Na, M+HCl+H, M+K, M+ACN+H

## Outputs

- Empirical compound (empCpd) annotations in JSON format
- Tab-delimited annotation table (e.g. this_test.tsv)
- Counts of explained (grouped, non-singleton) features per ionization mode before vs. after adduct expansion (summary e.g. yeast_extended_adduct_annotation_counts.csv)
- Khipu tree/grid visualizations (plain text or Jupyter-enhanced)

## How to apply

Start from a tab-delimited feature table (feature ID, m/z, rtime, then intensity columns) produced by asari 1.9.2; for the yeast demo, download ESI+ and ESI- tables from MassIVE MSV000087434 (Chen et al.) and apply the ESI- SNR > 100 quality filter as the cleaner demo set. Run khipu's mass-difference search with the built-in patterns (e.g. 1.0034 Da for 13C/12C isotopes and 22.9893 Da for Na+ adducts): pattern-matched pairs form a mass distance network that is partitioned into connected subnetworks, each converted to a directed acyclic graph (weeding out edges that violate the DAG/rooted-tree property) and aligned against the 2-tier adduct tree via two-step optimization; the M0 neutral mass is then inferred by linear regression on matched m/z values. To extend adduct coverage, overwrite the default search patterns in the Jupyter demo notebooks with the additional adducts from Chen et al. — in khipu, search patterns are deliberately separate from search functions, and per-adduct isotopic subnetworks are defined because all ions connected by isotopic edges belong to the same adduct. Count explained (grouped, non-singleton) features per mode before and after the adduct expansion and record empirical compound counts; the extension is expected to increase explained features while leaving empirical compound counts essentially unchanged (1,775 ESI+, 908 ESI-).

## Related tools

- **khipu** (Python 3 package performing the mass-difference search, mass-distance-network to DAG conversion, adduct-tree alignment, and neutral-mass regression; search patterns are user-overwritable to extend adduct coverage) — https://github.com/shuzhao-li/khipu
- **asari 1.9.2** (Preprocessing tool that generates the feature tables (and the ESI- SNR>100-filtered demo) that khipu annotates) — https://github.com/shuzhao-li/asari
- **Jupyter** (Demo notebooks under notebooks/ demonstrate how to supply custom isotope and adduct patterns when rerunning annotation)
- **mass2chem** (Provides the search functions used by khipu for pattern matching) — https://github.com/shuzhao-li-lab/mass2chem

## Examples

```
pip install khipu-metabolomics && khipu -i testdata/ecoli_pos.tsv -o this_test
```

## Evaluation signals

- Explained ESI+ features increase from 6,310 (default patterns) to 8,049 after adding the extended Chen et al. adducts
- Explained ESI- features (SNR>100-filtered demo) increase from 2,601 to 2,912
- Empirical compound counts remain essentially unchanged after adduct expansion: 1,775 (ESI+) and 908 (ESI-)
- Every subnetwork converts to a valid directed acyclic graph and rooted tree — edges violating the DAG/rooted-tree property are weeded out
- Output JSON and TSV files are produced and each empCpd carries an inferred M0 neutral mass from the regression model

## Limitations

- Khipu is a preannotation tool; complete annotation requires additional resources such as contaminants, authentic libraries, and tandem MS data.
- Annotation of untargeted metabolomics data, including isotopic tracing data, is still not fully solved.
- No benchmarking results yet; benchmarking is deferred to future work.
- Some ions may enter the initial network by mistakes or unresolved signals and are removed from the established khipu and sent off to form a new khipu.

## Evidence
