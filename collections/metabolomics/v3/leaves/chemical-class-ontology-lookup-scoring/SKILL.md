---
name: chemical-class-ontology-lookup-scoring
description: Use when NPLinker has produced candidate BGC–MS/MS spectrum links (e.g.,
  via standardised Metcalf scoring) and you need to filter false positives by matching
  genome-based and structure-based chemical class ontologies.
when_to_use_negative:
- Candidate links lack any CANOPUS or MolNetEnhancer structure-based class prediction
  and filter_missing_scores is enabled, since 1988 spectra without predictions would
  be discarded
- Large metabolites above the CANOPUS 850 Da mass cut-off, where Spec2Vec/MS2DeepScore-style
  methods may be better suited
- You need to discover links de novo rather than prioritise existing Metcalf-scored
  candidates — this score complements, not replaces, co-occurrence-based scoring
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NPClassScore
  role: Empirical class-matching score that looks up (GCF class, spectrum class) pairs
    in MIBiG-derived frequency tables and filters links below 0.25
  repo: github.com/NPLinker/nplinker
- name: NPLinker
  role: Platform in which NPClassScore is implemented, integrating Metcalf scoring,
    GNPS data, BiG-SCAPE GCFs, and scoring-table lookup
  repo: github.com/NPLinker/nplinker
- name: CANOPUS
  role: Predicts structure-based compound classes directly from MS/MS spectra via
    Sirius v4.9.3 on the GNPS mgf, providing the spectrum side of the class match
  repo: github.com/kaibioinfo/canopus_treemap
- name: MolNetEnhancer
  role: Alternative structure-based class annotator run externally on the GNPS platform
- name: BiG-SCAPE
  role: Groups BGCs into GCFs whose genome-based classes supply the BGC side of the
    class pair
- name: mibig_classifications
  role: Provides the code and MIBiG-based class matching scoring tables used for the
    lookup
  repo: github.com/louwenjjr/mibig_classifications
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s40168-022-01444-3
    title: Enhanced correlation-based linking of biosynthetic gene clusters to their
      metabolic products through chemical class matching
  source_content_hash: sha256:66a894c0761451264dd4f7cc996814761a3a3313d7a6165c4d0a0a9044a19a55
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-SCAPE
  - CANOPUS
  - MolNetEnhancer
  - NPClassScore
  - NPLinker
  - antiSMASH
  - mibig_classifications
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# chemical-class-ontology-lookup-scoring

## Summary

NPClassScore is an empirical chemical-class matching score that bridges genome-based BGC classes (from antiSMASH/BiG-SCAPE) and structure-based compound classes (from CANOPUS or MolNetEnhancer) using precomputed MIBiG 2.0-derived frequency tables, implemented in NPLinker to prioritise and filter candidate BGC–MS/MS links.

## When to use

Use when NPLinker has produced candidate BGC–MS/MS spectrum links (e.g., via standardised Metcalf scoring) and you need to filter false positives by matching genome-based and structure-based chemical class ontologies.

## When NOT to use

- Candidate links lack any CANOPUS or MolNetEnhancer structure-based class prediction and filter_missing_scores is enabled, since 1988 spectra without predictions would be discarded
- Large metabolites above the CANOPUS 850 Da mass cut-off, where Spec2Vec/MS2DeepScore-style methods may be better suited
- You need to discover links de novo rather than prioritise existing Metcalf-scored candidates — this score complements, not replaces, co-occurrence-based scoring

## Inputs

- Candidate BGC–MS/MS link list from standardised Metcalf scoring in NPLinker
- Genome-based class annotations of GCFs from BiG-SCAPE and antiSMASH outputs
- Structure-based class predictions from CANOPUS (via Sirius v4.9.3 on GNPS mgf) or MolNetEnhancer
- MIBiG-derived scoring tables (from MIBiG 2.0 json entries, github.com/louwenjjr/mibig_classifications)

## Outputs

- Scored and filtered BGC–MS/MS link list with NPClassScore values
- Links retained above the 0.25 NPClassScore cut-off, prioritising validated pairs

## How to apply

First annotate each candidate GCF with genome-based classes from BiG-SCAPE/antiSMASH output and each spectrum with structure-based classes from CANOPUS (run via Sirius v4.9.3 on the GNPS mgf) or MolNetEnhancer (run externally on the GNPS platform). Load the MIBiG-derived scoring tables mapping (genome-based class, structure-based class) pairs to relative frequencies, built from MIBiG 2.0 json entries classified with ClassyFire and NPClassifier. For each link that passed standardised Metcalf scoring (cut-off 2.5), look up the class pair in the tables and assign the maximum relative frequency as the NPClassScore. Filter out links scoring below the default cut-off of 0.25, optionally using the filter_missing_scores toggle to discard links without structure-based classes; the result is a reduced link list (on average 63% fewer links) that retains 96% of experimentally validated links.

## Related tools

- **NPClassScore** (Empirical class-matching score that looks up (GCF class, spectrum class) pairs in MIBiG-derived frequency tables and filters links below 0.25) — github.com/NPLinker/nplinker
- **NPLinker** (Platform in which NPClassScore is implemented, integrating Metcalf scoring, GNPS data, BiG-SCAPE GCFs, and scoring-table lookup) — github.com/NPLinker/nplinker
- **CANOPUS** (Predicts structure-based compound classes directly from MS/MS spectra via Sirius v4.9.3 on the GNPS mgf, providing the spectrum side of the class match) — github.com/kaibioinfo/canopus_treemap
- **MolNetEnhancer** (Alternative structure-based class annotator run externally on the GNPS platform)
- **BiG-SCAPE** (Groups BGCs into GCFs whose genome-based classes supply the BGC side of the class pair)
- **mibig_classifications** (Provides the code and MIBiG-based class matching scoring tables used for the lookup) — github.com/louwenjjr/mibig_classifications

## Examples

```
config = load_config('path/to/nplinker.toml'); npl.load_data(); link_graph = npl.get_links(npl.gcfs[:3], "metcalf")
```

## Evaluation signals

- Every retained link has an NPClassScore ≥ 0.25 and a corresponding class pair found in the MIBiG-derived scoring tables
- Link count reduction is consistent with reported behaviour (~63% fewer links vs co-occurrence alone; ~49% average decrease per GCF when not discarding missing-score links)
- Validated links are retained: ~96% of experimentally validated links across the three PoDP datasets survive filtering and are prioritised
- Output is a machine-readable table including NPClassScore values, and ranked positions of validated links before vs after filtering are recorded
- Spectra without CANOPUS or MolNetEnhancer predictions are handled consistently according to the filter_missing_scores toggle

## Limitations

- 1988 MS/MS spectra across the datasets lacked a CANOPUS or MolNetEnhancer prediction, and filtering discards their links
- CANOPUS coverage is limited by an 850 Da mass cut-off; larger metabolites are not well handled
- MolNetEnhancer cannot currently be run within NPLinker and must be run externally on the GNPS platform
- The Nocardia dataset showed low standardised Metcalf scores for validated links, requiring the threshold to be lowered from 2.5 to 2

## Evidence

- [discussion] Metcalf threshold and NPClassScore cut-off: "Standardised Metcalf scoring was used with a cut-off of 2.5, after which candidate links were filtered out if their NPClassScore score was below a cut-off of 0.25"
- [abstract] Link reduction vs co-occurrence: "the number of links is reduced by on average 63% as compared to using a co-occurrence-based strategy alone"
- [abstract] Validated link retention: "96% of experimentally validated links in these datasets are retained and prioritised when using NPClassScore"
- [readme] Scoring table construction principle: [evidence span withheld — anchor status: external_doc]
- [discussion] Where scoring tables live: "Code and scoring tables are available at github.com/louwenjjr/mibig_classifications"
- [discussion] CANOPUS input path: "Sirius v4.9.3 is used to run CANOPUS directly on the mgf file resulting from GNPS molecular networking"
