---
name: reaction-rule-based-pathway-prediction
description: Use when you have a paired transcriptomic–metabolomic dataset and want
  to anticipate biosynthetic reaction steps and candidate enzymes de novo, without
  a known 'bait' compound or enzyme.
when_to_use_negative:
- The pathway of interest hinges on stereoisomeric conversions, since MEANtools filters
  stereomeric rules and mass spectrometric data does not capture stereoisomers
- A reaction requires multiple substrates (e.g., p-coumaroyltyramine from tyrosine
  and p-coumaroyl-CoA), because RetroRules-based rules are designed for single-substrate
  reactions only
- You need confident metabolite annotation via MS/MS spectral libraries or molecular
  networking — no MS/MS data loading or spectral library/networking-based annotation
  is currently supported
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MEANtools
  role: 'Main pipeline: MAD rescaling, mutual-rank/exponential-decay edge weights,
    ClusterONE clustering, FC merging, and rule-based reaction anticipation'
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: RetroRules
  role: Source of general reaction rules (strict/medium/loose) mapping reactions to
    enzyme families; strict dataset (experimentally validated entries only) is used
    for anticipation
- name: LOTUS
  role: Metabolite structure database queried by mass and adduct at 20 ppm tolerance
    to assign molecular structures to mass features
  repo: github.com/IntegrativeBioinformaticsLab/meantools
- name: ClusterONE
  role: Clusters node types and edge weights into overlapping feature clusters (FCs)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pbio.3003307
    title: MEANtools integrates multi-omics data to identify metabolites and predict
      biosynthetic pathways
  source_content_hash: sha256:06310922f7e7a1495d48c77aba46cfdec136308fea375a7ef94c62c7ffe30c43
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterONE
  - LOTUS
  - MEANtools
  - RetroRules
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# reaction-rule-based-pathway-prediction

## Summary

Unsupervised prediction of biosynthetic pathway steps by integrating transcriptomic–metabolomic mutual-rank correlations with RetroRules reaction rules and LOTUS metabolite structures, as implemented in MEANtools and benchmarked on the Jeon 2020 tomato falcarindiol pathway (5 of 7 steps anticipated).

## When to use

Use when you have a paired transcriptomic–metabolomic dataset and want to anticipate biosynthetic reaction steps and candidate enzymes de novo, without a known 'bait' compound or enzyme.

## When NOT to use

- The pathway of interest hinges on stereoisomeric conversions, since MEANtools filters stereomeric rules and mass spectrometric data does not capture stereoisomers
- A reaction requires multiple substrates (e.g., p-coumaroyltyramine from tyrosine and p-coumaroyl-CoA), because RetroRules-based rules are designed for single-substrate reactions only
- You need confident metabolite annotation via MS/MS spectral libraries or molecular networking — no MS/MS data loading or spectral library/networking-based annotation is currently supported

## Inputs

- Paired transcriptomic-metabolomic dataset (e.g., Jeon 2020 tomato dataset)
- Combined RetroRules ruleset CSV (strict/medium/loose), strict subset recommended
- LOTUS database in SQLite format (downloaded 10/10/2023) or a user-defined metabolite database
- Mass feature list with m/z values and adduct candidates
- Gene expression matrix and transcript-to-Pfam/gene family mapping

## Outputs

- Predicted reaction steps connecting transcript-correlated mass features (Ni→j networks as DAGs)
- Candidate enzyme-gene assignments per reaction step with correlation statistics
- Feature clusters (FCs) and merged clusters from merge_clusters.py
- Pathway comparison table (e.g., pathway_steps_comparison.csv) tabulating predicted vs. published steps

## How to apply

Load the combined RetroRules ruleset CSV from Zenodo (record 15697913) and select the strict (experimentally validated entries only) rule set, since the loose set retains spurious reaction–Pfam links even after a Pfam cutoff of 6. Annotate mass features by identifying adducts and querying the LOTUS SQLite database at the default 20 ppm mass accuracy tolerance, then rescale input data with Median Absolute Deviation (MAD). Convert transcript–mass-feature Pearson correlations to mutual ranks, apply an exponential-decay function to obtain edge weights, cluster nodes and edge weights with ClusterONE into overlapping feature clusters (FCs), and merge FCs sharing mass features with merge_clusters.py. Run the reaction-anticipation step excluding gene–metabolite pairs with edge scores below 0.01, and remove cycles in DAGs by eliminating the weakest enzyme–metabolite correlation edge among metabolites at the same reaction distance. Compare predicted reaction steps against published pathway steps (e.g., the 7 falcarindiol steps of Jeon and colleagues 2020) and tabulate predicted/not-predicted calls per step.

## Related tools

- **MEANtools** (Main pipeline: MAD rescaling, mutual-rank/exponential-decay edge weights, ClusterONE clustering, FC merging, and rule-based reaction anticipation) — github.com/IntegrativeBioinformaticsLab/meantools
- **RetroRules** (Source of general reaction rules (strict/medium/loose) mapping reactions to enzyme families; strict dataset (experimentally validated entries only) is used for anticipation)
- **LOTUS** (Metabolite structure database queried by mass and adduct at 20 ppm tolerance to assign molecular structures to mass features) — github.com/IntegrativeBioinformaticsLab/meantools
- **ClusterONE** (Clusters node types and edge weights into overlapping feature clusters (FCs))

## Examples

```
wget https://zenodo.org/records/15697913/files/Combined_small_middle_big_datasets.csv
```

## Evaluation signals

- Benchmark count: on the Jeon 2020 tomato dataset under strict settings, MEANtools should anticipate 5 of the 7 published falcarindiol pathway steps
- The crepenynic acid → dehydrocrepenynic acid step must be predicted with gene Solyc12g100250.1 at correlation 0.744 and p-value 1.106E-10, matching the desaturase reported by Jeon and colleagues
- Strict vs. loose check: running with the loose ruleset should produce spurious reaction–Pfam links (e.g., AminoTran_1_2, Glyco_transf_20) that are absent with the strict set
- No gene–metabolite pairs with edge scores below 0.01 should appear in the Ni→j networks; cycles among same-distance metabolites should be resolved by removing the weakest enzyme–metabolism correlation edge
- Feature cluster sizes should be stable across treatment combinations in the Jeon 2020 dataset (as reported in S7 Fig)

## Limitations

- FC size may sometimes be insufficient for forming a complete biosynthetic pathway, and the current FC significance method was originally developed for co-expression datasets
- Changing ClusterONE's inbuilt size parameter also changes the clustering pattern of mass features and transcripts
- Mass spectrometer resolution directly impacts precision of detected m/z values and confidence of adduct-based matching
- Generating all product molecules for every RetroRules reaction would be time-consuming and computationally intensive
- Pathway reconstruction for PAL and THT FCs was hindered by stereoisomers and multi-substrate reactions

## Evidence

- [abstract] MEANtools correctly anticipated five out of seven steps of the characterized pathway: "MEANtools correctly anticipated five out of seven steps of the characterized pathway"
- [discussion] Gene–metabolite edge cutoff and rule-set choice: "Gene–metabolite pairs that show lower edge score than 0.01 are excluded in the Ni→j networks."
- [discussion] Cycle removal rule: "In instances where cycles occur among metabolites at the same reaction distance from the initial metabolite, the edge featuring the weakest enzyme-metabolite correlation is eliminated."
- [discussion] Mass tolerance: "By default, the system operates at a 20 ppm tolerance, which offers a balance between sensitivity and specificity."
- [discussion] Strict ruleset rationale: "These spurious links, coming from RetroRules, have been kept in the loose dataset after applying the Pfam cutoff of 6, which highlights the importance of using the strict rules dataset"
- [results] Benchmark enzyme prediction: [evidence span withheld — anchor status: not_found]
- [readme] Repo contents: "Read the documentation here:

https://meantools.readthedocs.io/en/latest/"
