---
name: gene-disease-association-mapping
description: Use when the goal is to confirm which genes are shared across a defined
  panel of intestine-related diseases, or to map specific genes (e.g. STK11, SMAD4/BMPR1A)
  to named syndromes using GutUDB's disease-gene association data.
when_to_use_negative:
- The question concerns non-intestinal diseases, since GutUDB's scope is 56 intestinal
  diseases across six species.
- The analysis needs raw omics profiles (e.g. the ~9 million generated profiles from
  eight omics data types) rather than curated disease-gene associations.
- A validated causal gene-disease relationship is required without further literature
  curation — GutUDB associations are curated records, not etiological proof.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3402
tools:
- name: GutUDB
  role: Source of the 260,790 curated disease-gene associations; queried via its Browse/Query/Visualization/Download
    modules to extract and cross-tabulate gene-disease links.
  repo: https://github.com/Ansanqi/GutUDB
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:b4a30c620ecd1fd8b0d38c788485a953e4b7a2febda617cb0b99c00f2efb3212
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ansanqi/GutUDB
  tools:
  - GutUDB
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3402
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# gene-disease-association-mapping

## Summary

Extracts and cross-tabulates disease-gene association records from GutUDB (260,790 curated associations across 56 intestinal diseases) to verify which genes are linked to a set of related diseases and to annotate disease-specific exemplar genes.

## When to use

Use when the goal is to confirm which genes are shared across a defined panel of intestine-related diseases, or to map specific genes (e.g. STK11, SMAD4/BMPR1A) to named syndromes using GutUDB's disease-gene association data.

## When NOT to use

- The question concerns non-intestinal diseases, since GutUDB's scope is 56 intestinal diseases across six species.
- The analysis needs raw omics profiles (e.g. the ~9 million generated profiles from eight omics data types) rather than curated disease-gene associations.
- A validated causal gene-disease relationship is required without further literature curation — GutUDB associations are curated records, not etiological proof.

## Inputs

- GutUDB disease-gene association dataset (subset covering the target intestinal diseases)
- List of target diseases (e.g. colon cancer, colorectal cancer, bleeding of the colon, rectal cancer, constipation, diarrhea, ileus, inflammatory bowel disease, small intestine cancer)
- List of candidate genes to verify (e.g. STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV)

## Outputs

- Verified gene-to-disease mapping table (gene x disease associations)
- Annotated list of disease-specific exemplar genes (e.g. STK11 -> Peutz-Jeghers syndrome; SMAD4/BMPR1A -> Juvenile Polyposis Syndrome)

## How to apply

Query the GutUDB disease-gene dataset for the target disease list and the candidate genes (e.g. STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV). Use the database's Browse/Query functionality to pull association records, then cross-tabulate the gene x disease matrix to confirm each candidate gene is linked to every disease in the panel. Annotate exemplar cases — STK11 for Peutz-Jeghers syndrome and SMAD4/BMPR1A for Juvenile Polyposis Syndrome — from the same association records. Write the verified gene-to-disease mapping, including exemplar annotations, to an output table; rationale is that GutUDB systematically embeds 260,790 disease-gene associations covering 56 intestinal diseases, making it a single source for shared-versus-disease-specific gene claims.

## Related tools

- **GutUDB** (Source of the 260,790 curated disease-gene associations; queried via its Browse/Query/Visualization/Download modules to extract and cross-tabulate gene-disease links.) — https://github.com/Ansanqi/GutUDB

## Examples

```
download GutUDB disease-gene associations (Download module), then in pandas: df = pd.read_csv('gutudb_disease_gene.tsv', sep='\t'); panel = ['colon cancer','colorectal cancer','bleeding of the colon','rectal cancer','constipation','diarrhea','ileus','inflammatory bowel disease','small intestine cancer']; shared = df[df.disease.isin(panel)].groupby('gene').disease.nunique(); shared[shared==9]
```

## Evaluation signals

- Every candidate gene (STK11, CFTR, BMPR1A, SMAD4, NOTCH1, PKD1, MLH1, MSH2, APC, MEFV) appears as associated with every disease in the nine-disease intestinal panel in the output table.
- Exemplar annotations match the curated records: STK11 linked to Peutz-Jeghers syndrome and SMAD4/BMPR1A linked to Juvenile Polyposis Syndrome.
- The total number of association records retrieved is consistent with GutUDB's reported 260,790 disease-gene associations for the covered disease set.
- Output table contains one row per gene with explicit disease membership and exemplar syndrome columns, and no missing gene entries.

## Limitations

- Association coverage is limited to GutUDB's curated content — 56 intestinal diseases, six species, and 58,970 genes from 11 intestinal tissue subtypes or 63 intestinal cell lines; diseases or genes outside this scope cannot be mapped.
- Shared association does not establish shared etiology; exemplar status (e.g. STK11 for Peutz-Jeghers syndrome) comes from the database's curation, not from this cross-tabulation itself.

## Evidence

- [abstract] 260,790 disease-gene associations are embedded in GutUDB: "In total, 260,790 disease‐gene associations were embedded in GutUDB"
- [abstract] GutUDB covers 56 intestinal diseases: "GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [abstract] Genes incorporated from intestinal tissues and cell lines: "GutUDB incorporated 58,970 genes derived from 11 subtypes of intestinal tissue or 63 intestinal cell lines"
- [abstract] Core database functionalities used to extract records: "four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB"
- [results] Shared gene list and exemplars surfaced by the task: [evidence span withheld — anchor status: not_found]
