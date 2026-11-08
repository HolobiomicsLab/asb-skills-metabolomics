---
name: empirical-compound-preannotation
description: Use when you have a peak/feature table (m/z, rtime, intensities) from
  LC-MS metabolomics or isotope-tracing experiments and want to group degenerate ions
  into empirical compounds before database annotation.
when_to_use_negative:
- 'Complete annotation: khipu is preannotation only; it does not incorporate contaminants,
  authentic compound libraries, or tandem MS data needed for final identification.'
- Input is already a set of identified compounds or peak groups rather than a raw
  feature table.
- Mass-only or MS/MS-centric workflows where ion relationships (isotopes/adducts)
  within a compound are not the target grouping.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: khipu
  role: Core Python package implementing the tree-structure preannotation algorithm
    on feature tables
  repo: https://github.com/shuzhao-li/khipu
- name: asari
  role: Upstream preprocessing tool that generates feature tables consumed by khipu;
    also uses khipu internally for preannotation
  repo: https://github.com/shuzhao-li/asari
- name: mass2chem
  role: Provides the search functions used to find isotope/adduct pattern matches
    among m/z values
  repo: https://github.com/shuzhao-li-lab/mass2chem
- name: metDataModel
  role: Defines the 'empirical compound' data model used for khipu output
  repo: https://github.com/shuzhao-li-lab/metDataModel
- name: Python 3
  role: Runtime language of the khipu package; pip-installable (khipu-metabolomics)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.2c05810
    title: Generalized Tree Structure to Annotate Untargeted Metabolomics and Stable
      Isotope Tracing Data
  source_content_hash: sha256:d00cd4f297a5463143cc9067670ad8bad03a48d372eb82970c1745ced1647343
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/shuzhao-li/khipu
  tools:
  - Jupyter
  - Python 3
  - asari
  - khipu
  - mass2chem
  - metDataModel
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# empirical-compound-preannotation

## Summary

Khipu-based preannotation of LC-MS feature tables into empirical compounds (trees) by linking isotope and adduct edges, inferring neutral M0 masses, and emitting JSON/tab-delimited outputs for downstream annotation.

## When to use

Use when you have a peak/feature table (m/z, rtime, intensities) from LC-MS metabolomics or isotope-tracing experiments and want to group degenerate ions into empirical compounds before database annotation.

## When NOT to use

- Complete annotation: khipu is preannotation only; it does not incorporate contaminants, authentic compound libraries, or tandem MS data needed for final identification.
- Input is already a set of identified compounds or peak groups rather than a raw feature table.
- Mass-only or MS/MS-centric workflows where ion relationships (isotopes/adducts) within a compound are not the target grouping.

## Inputs

- Tab-delimited LC-MS feature table (feature ID, m/z, rtime, intensity columns) from any preprocessing tool (e.g. asari)
- Ionization mode flag (pos or neg)
- Optional custom isotope/adduct search patterns for isotope-tracing or chemical-derivatization data
- Mass tolerance in ppm (--ppm) and retention-time tolerance (--rtol)

## Outputs

- Empirical compound (khipu tree) list in JSON format
- Tab-delimited empirical compound table
- Per-empCpd assignments of features to adduct/isotope grid slots and inferred neutral M0 masses

## How to apply

Start from a tab-delimited feature table (columns: feature ID, m/z, rtime, then intensities; specify intensity start/end columns). Search the feature list for isotope mass differences (e.g. 1.0034 Da for 13C/12C) and adduct differences (e.g. 22.9893 Da for Na+) to build a mass distance network; partition it into connected subnetworks, convert each to a directed acyclic graph where erroneous edges are weeded out, and align the abstracted network to a 2-tier adduct/isotope 'khipu grid' via two-step optimization (trunk of adducts, then isotopic branches). Assign ions to the grid and obtain the neutral M0 mass by linear regression over matched m/z values; mismatched ions are removed and can seed a new khipu. Run per ionization mode with default adducts (--mode pos or neg), tune --ppm mass precision and --rtol retention-time tolerance, and optionally supply custom isotope/adduct search patterns for isotope-tracing or derivatization experiments. Outputs are JSON and tab-delimited empirical compound files.

## Related tools

- **khipu** (Core Python package implementing the tree-structure preannotation algorithm on feature tables) — https://github.com/shuzhao-li/khipu
- **asari** (Upstream preprocessing tool that generates feature tables consumed by khipu; also uses khipu internally for preannotation) — https://github.com/shuzhao-li/asari
- **mass2chem** (Provides the search functions used to find isotope/adduct pattern matches among m/z values) — https://github.com/shuzhao-li-lab/mass2chem
- **metDataModel** (Defines the 'empirical compound' data model used for khipu output) — https://github.com/shuzhao-li-lab/metDataModel
- **Python 3** (Runtime language of the khipu package; pip-installable (khipu-metabolomics))

## Examples

```
khipu -i testdata/ecoli_pos.tsv -o this_test -m pos
```

## Evaluation signals

- Expected counts reproduce published benchmarks: e.g. khipu annotates 548 empirical compounds from 1,745 of 3,602 features on E. coli reverse-phase ESI+ data, and 1,775 (ESI+) and 908 (ESI-) empirical compounds on yeast MassIVE MSV000087434 data.
- Output files exist as a pair: <prefix>.json and <prefix>.tsv for the same run.
- Every empirical compound tree respects the khipu grid: isotope branches assigned to an adduct trunk; adduct edges and isotope edges separated before trunk optimization.
- Inferred neutral M0 masses are consistent with a linear regression over matched m/z values of the assigned grid ions.
- Ions that do not satisfy the rooted-tree/DAG constraints are excluded from the khipu and optionally re-seed new khipu instances.

## Limitations

- Benchmarking was deferred to future work; khipu performance on other platforms/datasets is not yet benchmarked in this paper.
- Annotation of untargeted metabolomics including isotopic tracing data is described as still not fully solved.
- If additional adducts are enabled, more features are explained (yeast ESI+ 6,310 → 8,049; ESI- 2,601 → 2,912) but the adduct list choice materially changes results.
- Some ions enter the initial network by mistake or are unresolved signals; these must be pruned from the khipu.
- The yeast ESI- demo relies on signal:noise >100 quality filtering for a cleaner table; noisier tables may degrade tree construction.
- Derivatization-experiment support (linking origin and derivatized compound trees by m/z shift) is described as the next step, not the current core capability.

## Evidence
