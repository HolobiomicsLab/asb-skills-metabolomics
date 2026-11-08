---
name: substructure-motif-frequency-analysis
description: Use when you have a GNPS molecular network and MS2LDA Mass2Motifs and
  need to quantify motif frequency across molecular families to find substructural
  diversity or common substructures linking known and unknown compounds.
when_to_use_negative:
- Input is already a feature-based molecular network with a MZmine-generated MGF —
  use the feature-based notebook (Mass2Motifs_2_Network_FeatureBased.ipynb) instead
  of the classical workflow
- No MS2LDA Mass2Motifs exist and no LDA experiment has been created — this skill
  requires motif-document relations as input, not raw spectra alone
- You only need chemical class annotation without substructure information — use the
  chemical-class-only pyMolNetEnhancer workflow instead
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MS2LDA
  role: Discovers Mass2Motifs from the MGF clustered spectra downloaded from GNPS,
    producing the motif-document summary table that is mapped onto the network
- name: pyMolNetEnhancer
  role: Maps MS2LDA Mass2Motifs onto the classical mass spectral molecular network
    using user-defined prob, overlap, and top parameters; produces .graphml, edges
    TSV, and nodes TSV outputs
  repo: https://github.com/madeleineernst/pyMolNetEnhancer
- name: GNPS
  role: Platform that creates the molecular network from which the clustered MGF and
    job ID are obtained
- name: Cytoscape
  role: Visualization of the .graphml output to color edges by shared Mass2Motifs
    and nodes by TopSharedMotifs
  repo: https://cytoscape.org/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo9070144
    title: 'MolNetEnhancer: Enhanced Molecular Networks by Integrating Metabolome
      Mining and Annotation Tools'
  source_content_hash: sha256:0b57627a560fe901220019ecb11346cdf28006b03236cd730ef3dd7a1268b299
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madeleineernst/pyMolNetEnhancer
  tools:
  - Cytoscape
  - GNPS
  - MS2LDA
  - pyMolNetEnhancer
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# substructure-motif-frequency-analysis

## Summary

Analyzes how frequently MS2LDA Mass2Motifs occur across molecular families in a GNPS molecular network, using pyMolNetEnhancer to map motif-to-document relations onto the network and identify high-frequency substructural handles (e.g. the amino sugar motif with frequency >70) shared across families.

## When to use

Use when you have a GNPS molecular network and MS2LDA Mass2Motifs and need to quantify motif frequency across molecular families to find substructural diversity or common substructures linking known and unknown compounds.

## When NOT to use

- Input is already a feature-based molecular network with a MZmine-generated MGF — use the feature-based notebook (Mass2Motifs_2_Network_FeatureBased.ipynb) instead of the classical workflow
- No MS2LDA Mass2Motifs exist and no LDA experiment has been created — this skill requires motif-document relations as input, not raw spectra alone
- You only need chemical class annotation without substructure information — use the chemical-class-only pyMolNetEnhancer workflow instead

## Inputs

- MGF file with clustered spectra downloaded from a GNPS molecular network job
- GNPS job ID
- MS2LDA job ID (or manually downloaded MS2LDA summary table of motif-document relations)
- User-defined mapping parameters: prob (default 0.01), overlap (default 0.3), top (default 5)

## Outputs

- Mass2Motifs_Edges_Classical.tsv (edges annotated with shared Mass2Motifs)
- Mass2Motifs_Nodes_Classical.tsv (nodes annotated with most shared motifs per molecular family, including TopSharedMotifs column)
- .graphml network file for Cytoscape visualization
- ms2lda_motif_summary.tsv (motif summary table with annotated motifs and frequencies)

## How to apply

Run MS2LDA (e.g. via ms2lda.org, experiment 912) on the MGF clustered spectra downloaded from GNPS, setting prob = 0.01 and overlap = 0.3 both in the web app's Experimental Options tab and in the pyMolNetEnhancer mapping step, because the exported summary table contains filtered motif-document relations using the thresholds set in the web app. Execute the Mass2Motifs_2_Network_Classical.ipynb notebook line by line, supplying your GNPS job ID and MS2LDA job ID, and set top = 5 to show the five most shared Mass2Motifs per molecular family (network component index). The workflow maps substructural information onto the classical mass spectral molecular network, producing edges colored by shared Mass2Motifs and nodes colored by the most shared motifs per family. Tally motif frequency across the network — e.g. in the Salinispora/Streptomyces marine sediment case, an amino sugar motif (fragments of N,N-dimethyl amino sugars such as dimethylamino-β-d-xylo-hexopyranoside and N,N-dimethyl-pyrrolosamine) reached a frequency of more than 70 and was found in all members of both the rosamicin and lomaiviticin molecular families plus two unknown families and several singletons, making it a usable handle for identifying known and potential novel natural products. Visualize the .graphml output in Cytoscape, coloring edges via 'Stroke Color' with 'interaction' as Column and 'Discrete Mapping' as Mapping Type, and coloring nodes via 'Image/Chart' using 'TopSharedMotifs'.

## Related tools

- **MS2LDA** (Discovers Mass2Motifs from the MGF clustered spectra downloaded from GNPS, producing the motif-document summary table that is mapped onto the network)
- **pyMolNetEnhancer** (Maps MS2LDA Mass2Motifs onto the classical mass spectral molecular network using user-defined prob, overlap, and top parameters; produces .graphml, edges TSV, and nodes TSV outputs) — https://github.com/madeleineernst/pyMolNetEnhancer
- **GNPS** (Platform that creates the molecular network from which the clustered MGF and job ID are obtained)
- **Cytoscape** (Visualization of the .graphml output to color edges by shared Mass2Motifs and nodes by TopSharedMotifs) — https://cytoscape.org/

## Examples

```
pip install pyMolNetEnhancer && python -c "from pyMolNetEnhancer import *; edges, nodes = Motif2Network(gnps_job_id, ms2lda_job_id, prob=0.01, overlap=0.3, top=5)"
```

## Evaluation signals

- The number of Mass2Motifs discovered matches the MS2LDA run configuration (e.g. 300 motifs in the Salinispora/Streptomyces marine sediment case study)
- The number of annotated motifs is consistent with the article's benchmark (40 motifs annotated with structural information from GNPS library matches or in silico tools such as NAP, DEREPLICATOR, VarQuest)
- A target motif of interest shows a frequency exceeding the expected threshold (e.g. the amino sugar motif with frequency >70) and is present in all members of the expected molecular families (e.g. both the rosamicin and lomaiviticin molecular families)
- The exported MS2LDA summary respects the prob = 0.01 and overlap = 0.3 thresholds set in the web app's Experimental Options tab
- TopSharedMotifs column in Mass2Motifs_Nodes_Classical.tsv shows at most 5 most shared motifs per molecular family (network component index), consistent with top = 5

## Limitations

- The MS2LDA summary table contains filtered motif-document relations using thresholds set in the web app, so changing Experimental Options after the run changes what is exported
- Depending on the size of the MS2LDA output file, a server connection timeout may occur when fetching it via job ID; manual download from ms2lda.org is the fallback
- Only motifs passing both the probability and overlap thresholds are retained, so low-probability or low-overlap motifs are invisible to the frequency analysis

## Evidence

- [other] MS2LDA analysis of the 5930-node Salinispora/Streptomyces network yielded 300 Mass2Motifs, 40 of which could be annotated: [evidence span withheld — anchor status: not_found]
- [other] An amino sugar-related Mass2Motif had a frequency of more than 70 across the network: [evidence span withheld — anchor status: not_found]
- [readme] pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks: "pyMolNetEnhancer is a python module integrating chemical class and substructure information within mass spectral molecular networks created through the Global Natural Products Social Molecular"
- [readme] The probability and overlap thresholds can be set within the ms2lda.org app as well under the Experimental Options tab: [evidence span withheld — anchor status: not_found]
- [readme] top: This parameter specifies how many most shared motifs per molecular family (network component index) should be shown: "top: This parameter specifies how many most shared motifs per molecular family (network component index) should be shown. Default is 5."
- [other] Run the pyMolNetEnhancer notebook Mass2Motifs_2_Network_Classical.ipynb with the GNPS job ID and MS2LDA job ID: [evidence span withheld — anchor status: not_found]
