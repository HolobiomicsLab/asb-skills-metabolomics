---
name: lcms-feature-grouping
description: Use when you have a feature table from LC-MS untargeted metabolomics
  or stable isotope tracing and need to group degenerate ions (isotopes/adducts) into
  empirical compounds with inferred neutral masses, without prior formula knowledge.
when_to_use_negative:
- Input is raw LC-MS data (mzML/CDF) rather than a preprocessed feature table — use
  asari or another peak-picking tool first to produce the feature table.
- Complete annotation is needed including contaminants, authentic spectral libraries,
  and tandem MS confirmation — khipu provides pre-annotation only, not full compound
  identification.
- The goal is chromatographic peak deconvolution or retention-time alignment across
  samples; khipu operates on the feature table's ions, not on raw chromatograms.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: khipu
  role: Standalone CLI and Python library that performs the mass-distance-network-to-tree
    grouping and neutral-mass regression on the feature table
  repo: https://github.com/shuzhao-li/khipu
- name: asari
  role: Preprocessing tool that produces the feature table input; also uses khipu
    internally for pre-annotation
  repo: https://github.com/shuzhao-li/asari
- name: mass2chem
  role: Underlying library providing the mass-difference search functions used by
    khipu
  repo: https://github.com/shuzhao-li-lab/mass2chem
- name: metDataModel
  role: Defines the 'empirical compound' data model used for downstream annotation
  repo: https://github.com/shuzhao-li-lab/metDataModel
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:be33aa09d6e05401136ed54a683f361b8ac235d63e0855911a1cfcb0f2fb1487
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shuzhao-li/khipu
  tools:
  - Python 3
  - asari
  - khipu
  - mass2chem
  - metDataModel
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# LC-MS feature grouping

## Summary

Groups raw LC-MS features into empirical compounds (khipus) by connecting ions via isotope and adduct mass-difference edges (e.g. 1.0034 for 13C/12C, 22.9893 for Na+) within a tree structure, inferring neutral M0 masses by regression. Provides degenerate-ion pre-annotation for untargeted metabolomics and stable isotope tracing data.

## When to use

Use when you have a feature table from LC-MS untargeted metabolomics or stable isotope tracing and need to group degenerate ions (isotopes/adducts) into empirical compounds with inferred neutral masses, without prior formula knowledge.

## When NOT to use

- Input is raw LC-MS data (mzML/CDF) rather than a preprocessed feature table — use asari or another peak-picking tool first to produce the feature table.
- Complete annotation is needed including contaminants, authentic spectral libraries, and tandem MS confirmation — khipu provides pre-annotation only, not full compound identification.
- The goal is chromatographic peak deconvolution or retention-time alignment across samples; khipu operates on the feature table's ions, not on raw chromatograms.

## Inputs

- Tab-delimited LC-MS feature table with columns: feature ID, m/z, rtime, followed by intensity columns (start and end column of intensity data must be specified if not default)
- Ionization mode (pos or neg)
- Optional: custom isotope/adduct search patterns for isotope tracing or derivatization experiments

## Outputs

- Khipu transparent JSON file of empirical compounds (empCpd format)
- Tab-delimited table of annotated empirical compounds
- Per-feature assignment to empirical compounds (grouped features vs singletons)

## How to apply

Feed a tab-delimited feature table (columns: feature ID, m/z, rtime, then intensities) produced by asari or another preprocessing tool into khipu with the ionization mode (--m pos|neg), mass precision in ppm, and retention-time tolerance. Khipu searches the feature list for isotope/adduct mass-difference patterns, connects matched pairs into a mass-distance network, partitions it into connected subnetworks, converts each to a directed acyclic graph (weeding out edges that break DAG/rooted-tree constraints), and optimizes assignment of isotopic branches onto an adduct trunk against a 'khipu grid' (e.g. M+H, M+NH4, M+Na, M+K, M+ACN+H with up to 6 13C isotopes). Neutral mass of the M0 compound is obtained by linear regression over matched m/z values. Ions entering the initial network by mistake are removed and sent to form a new khipu. For isotope tracing, override the default isotope patterns with experiment-specific search patterns as shown in the demo notebooks.

## Related tools

- **khipu** (Standalone CLI and Python library that performs the mass-distance-network-to-tree grouping and neutral-mass regression on the feature table) — https://github.com/shuzhao-li/khipu
- **asari** (Preprocessing tool that produces the feature table input; also uses khipu internally for pre-annotation) — https://github.com/shuzhao-li/asari
- **mass2chem** (Underlying library providing the mass-difference search functions used by khipu) — https://github.com/shuzhao-li-lab/mass2chem
- **metDataModel** (Defines the 'empirical compound' data model used for downstream annotation) — https://github.com/shuzhao-li-lab/metDataModel

## Examples

```
khipu -i testdata/ecoli_pos.tsv -o this_test
```

## Evaluation signals

- On the E. coli ESI+ test table (ecoli_pos.tsv, 3,602 features), expect exactly 548 empirical compounds, 1,745 features grouped, 1,857 singletons, and 445 empirical compounds with multiple isotopes
- Output must contain both this_test.json (khipu empCpd JSON) and this_test.tsv (tab-delimited table) with matching empirical compound counts
- Check that no grouped empirical compound violates the rooted-tree/DAG constraint — redundant nodes must have been removed
- For isotope tracing data, verify that the neutral M0 mass inferred by regression matches known labeled compound formulas (e.g. acetylspermidine C9H21N3O with 9 carbons at neutral mass 187.1686)
- Singleton count should be roughly total features minus grouped features, confirming the network partition did not silently drop features

## Limitations

- Complete annotation requires additional resources (contaminant lists, authentic libraries, tandem MS data) beyond khipu's pre-annotation.
- Some ions may enter the initial mass-distance network by mistake or as unresolved signals and must be removed during tree construction.
- The khipu grid is restricted to specific adduct types and isotope counts; extending it requires manual pattern search beyond the core construction.
- The method is benchmark-deferred; benchmarking and improvements are noted as future work.
- Isotopic tracing annotation is still not fully solved; the default patterns may not suit all experiments without custom patterns.

## Evidence
