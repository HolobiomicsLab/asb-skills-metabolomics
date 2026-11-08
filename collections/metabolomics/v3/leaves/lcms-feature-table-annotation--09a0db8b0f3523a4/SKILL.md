---
name: lcms-feature-table-annotation
description: Use when you have a tab-delimited feature table (feature ID, m/z, rtime,
  intensities) from LC-MS data — e.g. from asari v1.9.2 — and need to group degenerate
  ions into empirical compounds and infer neutral masses before database annotation,
  including stable isotope tracing.
when_to_use_negative:
- You need complete annotation including metabolite names — khipu is pre-annotation
  only; full annotation still requires contaminants, authentic libraries, and tandem
  MS data
- You only have raw LC-MS files — preprocessing (e.g. asari v1.9.2) must run first
  to produce the feature table
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: khipu
  role: 'Core algorithm: Weavor and Khipu classes convert the mass distance network
    into optimal 2-tier adduct/isotope trees and infer M0 neutral mass by regression'
  repo: https://github.com/shuzhao-li/khipu
- name: asari 1.9.2
  role: Upstream preprocessing that generates the feature tables consumed by khipu;
    asari uses khipu internally for preannotation
  repo: https://github.com/shuzhao-li/asari
- name: mass2chem
  role: Provides the mass-difference search functions used to find isotope/adduct
    pattern-matched feature pairs
  repo: https://github.com/shuzhao-li-lab/mass2chem
- name: metDataModel
  role: Defines the 'empirical compound' data model used by khipu
  repo: https://github.com/shuzhao-li-lab/metDataModel
- name: networkx
  role: Supports graph operations when building and pruning the mass distance network
    DAG
- name: treelib
  role: Aids tree visualization of khipu structures
- name: Jupyter
  role: Notebooks under notebooks/ demonstrate algorithm reuse, custom isotope/adduct
    patterns, and enhanced visualization
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:8125fc535500d7321324bfa948ffdf5541cc946f2762aa018d34e5d3dcb5f5ab
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shuzhao-li/khipu
  tools:
  - Jupyter
  - Python 3
  - asari 1.9.2
  - khipu
  - mass2chem
  - metDataModel
  - networkx
  - treelib
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# lcms-feature-table-annotation

## Summary

Pre-annotation of LC-MS feature tables into khipu trees ('empirical compounds') by connecting isotope and adduct edges into a mass distance network, converting it to a directed acyclic graph, and inferring the M0 neutral mass by regression — the generalized tree structure of Li and Zheng (2023).

## When to use

Use when you have a tab-delimited feature table (feature ID, m/z, rtime, intensities) from LC-MS data — e.g. from asari v1.9.2 — and need to group degenerate ions into empirical compounds and infer neutral masses before database annotation, including stable isotope tracing.

## When NOT to use

- You need complete annotation including metabolite names — khipu is pre-annotation only; full annotation still requires contaminants, authentic libraries, and tandem MS data
- You only have raw LC-MS files — preprocessing (e.g. asari v1.9.2) must run first to produce the feature table

## Inputs

- Tab-delimited feature table (columns: feature ID, m/z, rtime, intensities; e.g. asari v1.9.2 output)
- Ionization mode (pos or neg)
- Mass precision in ppm and RT tolerance (optional parameters)
- Optional custom isotope/adduct search patterns for isotope tracing or chemical labeling experiments

## Outputs

- JSON file of annotated empirical compounds (khipu trees), e.g. this_test.json
- Tab-delimited table of annotated features, e.g. this_test.tsv

## How to apply

1. Input a tab-delimited feature table whose first columns are feature ID, m/z, rtime followed by intensities (specify start/end intensity columns with -s/-e). 2. Choose ionization mode ('pos' or 'neg' via -m), mass precision in ppm (--ppm), and RT tolerance (--rtol, unit depends on the preprocessing tool). 3. Khipu searches the feature list for isotope patterns (e.g. 1.0034 Da 13C/12C steps) and adduct patterns (M+H 1.007276, M+NH4 18.033826, M+Na 22.989276, M+HCl+H 36.983976, M+K 38.963158, M+ACN+H 42.033825), connects matched pairs into a mass distance network, and partitions it into connected subnetworks. 4. Each subnetwork is pruned of redundant nodes and converted to a directed acyclic graph, then aligned to the 2-tier adduct/isotope 'khipu grid' by two-step optimization (trunk of adducts, branches of isotope edges); the neutral mass of M0 is obtained by linear regression on matched m/z values. 5. Extend the initial grid with additional adducts after core construction — in the yeast data this raised explained ESI+ features from 6,310 to 8,049 and ESI- from 2,601 to 2,912. 6. Export annotated empirical compounds in JSON and tab-delimited formats; the JSON format is deliberately transparent for interoperability with downstream data-science tools.

## Related tools

- **khipu** (Core algorithm: Weavor and Khipu classes convert the mass distance network into optimal 2-tier adduct/isotope trees and infer M0 neutral mass by regression) — https://github.com/shuzhao-li/khipu
- **asari 1.9.2** (Upstream preprocessing that generates the feature tables consumed by khipu; asari uses khipu internally for preannotation) — https://github.com/shuzhao-li/asari
- **mass2chem** (Provides the mass-difference search functions used to find isotope/adduct pattern-matched feature pairs) — https://github.com/shuzhao-li-lab/mass2chem
- **metDataModel** (Defines the 'empirical compound' data model used by khipu) — https://github.com/shuzhao-li-lab/metDataModel
- **networkx** (Supports graph operations when building and pruning the mass distance network DAG)
- **treelib** (Aids tree visualization of khipu structures)
- **Jupyter** (Notebooks under notebooks/ demonstrate algorithm reuse, custom isotope/adduct patterns, and enhanced visualization)

## Examples

```
khipu -i testdata/ecoli_pos.tsv -o this_test
```

## Evaluation signals

- Yeast data sets (MassIVE MSV000087434): ESI+ annotation should yield 1,775 empirical compounds and ESI- (signal:noise >100 filtered) should yield 908 — fewer than 2,000 compounds reliably measured
- E. coli reverse-phase ESI+ data with 3,602 LC-MS features: khipu should annotate 548 empirical compounds from 1,745 features
- For the U-13C E. coli data, the tree with inferred neutral mass 187.1686 must match acetylspermidine (C9H21N3O, 9 carbons), with the highest isotope peak at 13C/12C*9 (M9) in labeled samples and protonated plus Na/K adduct ions
- Output must contain two files per run: JSON (this_test.json) and tab-delimited (this_test.tsv), with the JSON in khipu's transparent format
- Erroneous edges that do not satisfy DAG and rooted-tree constraints should be weeded out; stray ions removed from a khipu are recycled into a new khipu instance

## Limitations

- Annotation of untargeted metabolomics data, including isotopic tracing data, is still not fully solved — khipu is pre-annotation, not complete annotation
- Benchmarking results were deferred to future work
- Prior network-approach tools (and the deficit khipu addresses) depended on assumed base ions or formulas
- Different applications (isotope tracing, chemical labeling, derivatization) require different parameters; users must overwrite default isotope search patterns per experiment
- Complete annotation still requires additional resources such as contaminants, authentic libraries, and tandem mass spectrometry data

## Evidence
