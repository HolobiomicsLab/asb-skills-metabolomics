---
name: isotopologue-pattern-interpretation
description: Use when a preprocessed feature table contains unexplained ion groups
  whose members differ by isotope or adduct mass shifts (e.g. 1.0034 Da 13C/12C, 22.9893
  Da Na+), or when annotating U-13C-labeled tracing samples.
when_to_use_negative:
- Input is raw LC-MS data rather than a preprocessed feature table — khipu expects
  asari-style feature tables, so preprocessing with asari 1.9.2 must come first.
- Complete metabolite identification is required, including confirmation against authentic
  libraries, contaminant handling, or tandem MS data — khipu provides preannotation
  only and these must be handled downstream.
- The goal is de novo formula assignment without any prior isotope/adduct pattern
  knowledge — the method starts by searching user-definable mass-difference patterns
  and is one of the tools whose annotation of isotopic tracing data remains 'not fully
  solved'.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: khipu
  role: 'Core algorithm: builds mass distance networks, converts them to DAGs and
    2-tier adduct/isotope trees, infers neutral mass by regression, exports empirical
    compounds in JSON'
  repo: https://github.com/shuzhao-li/khipu
- name: asari 1.9.2
  role: Preprocessing tool that generated the feature tables (ecoli_pos.tsv, yeast
    ESI+/ESI-) used as khipu input; asari uses khipu internally for preannotation
  repo: https://github.com/shuzhao-li/asari
- name: mass2chem
  role: Provides the search functions khipu uses for isotope/adduct pattern matching
  repo: https://github.com/shuzhao-li-lab/mass2chem
- name: networkx
  role: Supports the graph operations underlying mass distance network construction
    and DAG conversion
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:253d802621cbcc36cf15d116173ef4486f55662bab0af1813a8df57713749ec7
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
  - networkx
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# isotopologue-pattern-interpretation

## Summary

Groups LC-MS features into isotopologue/adduct ion trees ('empirical compounds') using khipu's generalized tree structure, inferring the neutral mass of M0 by regression; works for regular untargeted metabolomics and stable-isotope tracing data.

## When to use

Use when a preprocessed feature table contains unexplained ion groups whose members differ by isotope or adduct mass shifts (e.g. 1.0034 Da 13C/12C, 22.9893 Da Na+), or when annotating U-13C-labeled tracing samples.

## When NOT to use

- Input is raw LC-MS data rather than a preprocessed feature table — khipu expects asari-style feature tables, so preprocessing with asari 1.9.2 must come first.
- Complete metabolite identification is required, including confirmation against authentic libraries, contaminant handling, or tandem MS data — khipu provides preannotation only and these must be handled downstream.
- The goal is de novo formula assignment without any prior isotope/adduct pattern knowledge — the method starts by searching user-definable mass-difference patterns and is one of the tools whose annotation of isotopic tracing data remains 'not fully solved'.

## Inputs

- Tab-delimited feature table (feature ID, m/z, rtime, intensity columns; e.g. testdata/ecoli_pos.tsv from U-13C-labeled E. coli, Cambridge Isotopes MSK-CRED-DD-KIT)
- Ionization mode (pos or neg)
- Optional user-supplied isotope/adduct search patterns for the specific labeling experiment
- Parameters: ppm mass tolerance, retention-time tolerance (rtol), start/end intensity columns

## Outputs

- Annotated empirical compounds (khipu trees) in JSON format
- Same annotation in tab-delimited TSV format
- Per-tree inferred M0 neutral mass (via regression), isotope branch and adduct trunk assignments

## How to apply

Start from a tab-delimited feature table (columns: feature ID, m/z, rtime, then intensity columns) produced by asari 1.9.2 or any preprocessor. Run khipu in the correct ionization mode: it searches pattern-matched feature pairs using default isotope patterns (e.g. 1.0034 Da for 13C/12C) and adducts (e.g. 22.9893 Da for Na+), connects them into a mass distance network, partitions it into connected subnetworks, and converts each subnetwork to a directed acyclic graph (weeding out erroneous edges that cannot satisfy DAG/rooted-tree constraints). Each subnetwork is assembled into a tree by separating isotope edges into branches grouped by shared nodes (one branch per adduct type), optimizing an adduct 'trunk' to maximize explained nodes, and re-aligning isotopes to the khipu grid (M0 through 13C/12C*6 by default). The neutral mass of M0 is then obtained by linear regression from the matched m/z values. For isotope-tracing experiments, overwrite the default patterns with user-supplied search patterns matched to the labeling scheme — e.g. for U-13C E. coli data, expect the highest isotope peak at 13C/12C*9 (M9) for a 9-carbon compound such as acetylspermidine (C9H21N3O, inferred neutral mass 187.1686) carrying protonated plus Na and K adducts.

## Related tools

- **khipu** (Core algorithm: builds mass distance networks, converts them to DAGs and 2-tier adduct/isotope trees, infers neutral mass by regression, exports empirical compounds in JSON) — https://github.com/shuzhao-li/khipu
- **asari 1.9.2** (Preprocessing tool that generated the feature tables (ecoli_pos.tsv, yeast ESI+/ESI-) used as khipu input; asari uses khipu internally for preannotation) — https://github.com/shuzhao-li/asari
- **mass2chem** (Provides the search functions khipu uses for isotope/adduct pattern matching) — https://github.com/shuzhao-li-lab/mass2chem
- **networkx** (Supports the graph operations underlying mass distance network construction and DAG conversion)

## Examples

```
khipu -i testdata/ecoli_pos.tsv -o this_test
```

## Evaluation signals

- Inferred neutral mass of a selected tree must match the expected compound formula — e.g. 187.1686 for acetylspermidine (C9H21N3O, 9 carbons).
- In U-13C-labeled samples, the tree's highest isotope peak should appear at 13C/12C*N where N equals the compound's carbon count (M9 for 9-carbon acetylspermidine).
- Isotope edges must group by shared nodes into branches, each belonging to one adduct type, before trunk optimization; erroneous edges are those that do not satisfy DAG and rooted-tree constraints and should be removed.
- Benchmark against reported counts: on ecoli_pos.tsv (3,602 features) khipu annotated 548 empirical compounds from 1,745 features; on yeast data it yielded 1,775 (ESI+) and 908 (ESI-) empirical compounds, with explained features rising from 6,310 to 8,049 (ESI+) and 2,601 to 2,912 (ESI-) when extended adducts are added.
- Output JSON must conform to khipu's 'empirical compound' data model (metDataModel) and round-trip alongside the tab-delimited file.

## Limitations

- Annotation of untargeted metabolomics data, including isotopic tracing data, is still not fully solved; khipu is explicitly positioned as preannotation.
- Some ions enter the initial network by mistake or as unresolved signals; they are removed from the established khipu and sent off to form a new khipu.
- The default khipu grid covers only up to 13C/12C*6; longer isotopologue series (e.g. M9) require user-supplied search patterns matched to the experiment.
- Many prior network-approach tools depend on assumed base ions or formulas — care is needed not to over-assign relationships between ions.

## Evidence
