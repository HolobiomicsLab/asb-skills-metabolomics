---
name: adduct-isotope-derivative-enumeration
description: Use when an untargeted LC/MS feature table (m/z, retention time, p-value)
  must be mapped to metabolites without MS/MS spectra, by enumerating isotopic and
  adduct forms (e.g. M+H[1+], M(C13)+H[1+], M+Na[1+], M−H[−]) from neutral compound
  formulas in a genome-scale metabolic model.
when_to_use_negative:
- The goal is definitive metabolite identification from MS/MS fragmentation patterns
  or coelution of isotopically labeled known references — mummichog explicitly does
  not replace tandem mass spectrometry in metabolite identification.
- Features are quantified and identified peaks from a targeted assay rather than an
  untargeted high-resolution profiling feature table; enumeration of isotope/adduct
  space is unnecessary and can misassign identities.
- Input features are raw or non-well-formatted spectra rather than a quality-controlled
  feature table — upstream quantification and feature selection are expected to be
  complete, as mummichog consumes an already-selected significant feature list and
  reference list.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: mummichog
  role: Core package implementing the m/z feature to possibly-matched-metabolite enumeration,
    isotope/adduct matching and confidence rating that feeds its own pathway and network
    module analyses
  repo: https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- name: metDataModel
  role: Provides the data structures for metabolites, annotation tables and metabolic
    models that the enumeration operates on
- name: JMS
  role: Supplies reusable data structures for peaks, compounds and efficient mass
    search; used to structure annotations that the enumeration can consume, and holds
    model-conversion notebooks
  repo: https://github.com/shuzhao-li-lab/JMS/tree/main/notebooks
- name: Azimuth DB
  role: Chemical database for biology containing metabolic models and compound formulas
    that supply the neutral masses and formulas enumerated
- name: cloud-mummichog
  role: Server/worker RESTful implementation exposing the same enumeration as a web
    service
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1003123
    title: Predicting Network Activity from High Throughput Metabolomics
  source_content_hash: sha256:965185028c0f90673ef4f7e08b6c0ab212e5d4968bb49988e69ee0387801f32a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Azimuth DB
  - JMS
  - cloud-mummichog
  - metDataModel
  - mummichog
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# adduct-isotope-derivative-enumeration

## Summary

Enumerates the isotopic derivatives and adducts of every candidate neutral compound formula from a metabolic model, then matches these masses against untargeted LC/MS m/z features to assign tentative metabolite predictions with confidence ratings, forming the input-metabolite layer that mummichog's network module and pathway analyses build upon.

## When to use

Use when an untargeted LC/MS feature table (m/z, retention time, p-value) must be mapped to metabolites without MS/MS spectra, by enumerating isotopic and adduct forms (e.g. M+H[1+], M(C13)+H[1+], M+Na[1+], M−H[−]) from neutral compound formulas in a genome-scale metabolic model.

## When NOT to use

- The goal is definitive metabolite identification from MS/MS fragmentation patterns or coelution of isotopically labeled known references — mummichog explicitly does not replace tandem mass spectrometry in metabolite identification.
- Features are quantified and identified peaks from a targeted assay rather than an untargeted high-resolution profiling feature table; enumeration of isotope/adduct space is unnecessary and can misassign identities.
- Input features are raw or non-well-formatted spectra rather than a quality-controlled feature table — upstream quantification and feature selection are expected to be complete, as mummichog consumes an already-selected significant feature list and reference list.

## Inputs

- Feature table of m/z, retention time and p-value per feature (e.g. TSV such as ineurons_ttest_1127.tsv)
- Genome-scale metabolic model with list_of_pathways, list_of_reactions, list_of_compounds including neutral chemical formulas (JSON)
- Optional annotation table on features (metDataModel JSON from authentic standards or MS/MS, e.g. empCpds_with_annotations.json)
- Ionization mode (positive or negative) and mass tolerance

## Outputs

- Per-feature list of possibly matched metabolites with isotopic derivatives and adducts identified (ion_relation strings like M+H[1+], M+Na[1+])
- Confidence ratings on each metabolite prediction (high confidence when both M+H[1+] and M(C13)+H[1+] forms are present)
- Input-metabolite layer used by mummichog for network module extraction and pathway enrichment (result tables, result.html, JSON pathway/module analysis strings)

## How to apply

Load the reference metabolic model (e.g. KEGG-, UCSD Recon1- or BioCyc-based; supplied as list_of_pathways, list_of_reactions, list_of_compounds via metDataModel) ensuring each compound carries a correct neutral formula (never a salt formula, since salts form in both biology and mass spectrometry). For each m/z feature in the user-supplied feature table, compute all possibly matched metabolites by generating the isotopic derivatives and adducts of every candidate neutral compound formula within the instrumental mass tolerance, applying the confidence rating system: a metabolite prediction carrying both the single-charged M+H[1+] form and the M(C13)+H[1+] form is assigned high confidence. Optionally enforce the presence of the M+H[+] form (positive mode) or M−H[−] form (negative mode) as a stricter filter. Where available, consume an external annotation table (from authentic standards or MS/MS via metDataModel/JMS structures) to boost or validate identity assignments. Feed the resulting possibly-matched metabolites into mummichog's module and pathway enrichment steps; a single m/z feature may legitimately match several metabolites, which is accounted for downstream by the smaller-of correction in Fisher's exact test and by module-activity scoring.

## Related tools

- **mummichog** (Core package implementing the m/z feature to possibly-matched-metabolite enumeration, isotope/adduct matching and confidence rating that feeds its own pathway and network module analyses) — https://github.com/shuzhao-li/mummichog/blob/master/mummichog/models.py
- **metDataModel** (Provides the data structures for metabolites, annotation tables and metabolic models that the enumeration operates on)
- **JMS** (Supplies reusable data structures for peaks, compounds and efficient mass search; used to structure annotations that the enumeration can consume, and holds model-conversion notebooks) — https://github.com/shuzhao-li-lab/JMS/tree/main/notebooks
- **Azimuth DB** (Chemical database for biology containing metabolic models and compound formulas that supply the neutral masses and formulas enumerated)
- **cloud-mummichog** (Server/worker RESTful implementation exposing the same enumeration as a web service)

## Examples

```
python3 -m mummichog.main -i tests/ineurons_ttest_1127.tsv -j testneuron -a tests/empCpds_with_annotations.json -d .
```

## Evaluation signals

- A metabolite prediction assigned high confidence must show co-presence of both the M+H[1+] and M(C13)+H[1+] forms among the matched features.
- Every enumerated mass must trace back to a neutral compound formula in the loaded model (not a salt formula); check that neutral formula mass (e.g. 268.08077 for C10H12N4O5) matches the enumerated isotopic/adduct masses.
- Each matched feature should be linked to its matched metabolite with a specific ion_relation (e.g. M+H[1+], M+Na[1+]) and retention-time coherence (e.g. feature FT1705 at 99.90 s and FT1876 at 99.53 s for the same M0 isotope cluster).
- The number of possibly matched metabolites should exceed the number of features only where a feature legitimately matches multiple metabolites, and downstream Fisher's exact test must apply the smaller-of (overlap metabolites vs features) correction so overlap counts are not inflated.
- On the moDC/YF17D validation, the pipeline must reproduce a predicted activity network of 60 metabolites, of which 19 were not found in the original annotation.

## Limitations

- A single m/z feature may match multiple metabolites in the same module/pathway, inflating the overlap number — the smaller-of correction in Fisher's exact test mitigates but does not eliminate this.
- Features from high-resolution profiling experiments by far exceed current annotations in metabolite databases, so most features remain unmatched (de facto filtering).
- Compound formulas may be missing in a genome-scale metabolic model and must be looked up via compound identifiers; identifier alignment with other data needs a translation module if not using mummichog-supplied models.
- The current mummichog version does not incorporate a statistical matrix from the feature selection step, nor mass flow balance of metabolic reactions; quantification and feature selection upstream can still be problematic.
- Metabolic models need improvement on lipid annotation, physiological context and tissue specificity, so lipid-related identifications may be underrepresented.

## Evidence

- [results] Enumerates isotopic derivatives and adducts from neutral compound formulas for each m/z feature, applying confidence rating: "Mummichog collects these metabolites, and look up all their isotopic derivatives and adducts in . A confidence rating system is applied to filter for qualified metabolites."
- [results] Co-presence of M+H[1+] and M(C13)+H[1+] forms confers high confidence: "if both the single-charged  form M+H[1+] and the  form M(C13)+H[1+] are present, this metabolite prediction carries a high confidence."
- [results] Optional enforcement of M+H[+] form in positive mode or M−H[−] in negative mode: "There is an option in mummichog to enforce the presence of M+H[+] form (for positive mode, M−H[−] for negative mode) in metabolite prediction."
- [readme] README: salt formula must not be used; neutral formula is required for adduct calculation: "If compound formulas are provided in a model, be sure to have correct neutral mass and not a salt format. Salts are formed in both biological systems and in mass spectrometry."
- [results] README: mummichog computes all possibly matched metabolites from feature tables: "mummichog computes all possibly matched metabolites, and searches the reference metabolic network for all the modules that can be formed by these tentative metabolites."
