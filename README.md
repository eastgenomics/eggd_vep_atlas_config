# eggd_vep_atlas_config
JSON configuration file for VEP in the atlas pipeline

## What does the JSON config do?

Configuration file required to annotate a vcf using [Variant Effect Predictor](https://github.com/Ensembl/ensembl-vep) implementation [eggd_vep](https://github.com/eastgenomics/eggd_vep).

A variable level of annotation can be achieved by different combinations of custom annotations and vep plugins, in addition to the required VEP resources.

## What does the JSON config version contain?

This json file provides information about annotations,plugins, required fields and the genome version.

* Genome build: GRCh38
* VEP required files:
  * {placeholder as this will depend on VEP version}
* Custom Annotation sources:
  * clinvar_{version}.vcf.gz
  * gnomad.exomes.v4.1.sites.all.trimmed_normalised_decomposed_PASS.no_chr.vcf.bgz
  * gnomad.genomes.v4.1.sites.all.trimmed_normalised_decomposed_PASS.no_chr.vcf.bgz
  * {placeholder for GENIE}
  * {placeholder for prev counts}
* Plugin annotations:
  * SpliceAI
    * spliceai_scores.masked.snv.hg38.vcf.gz
    * spliceai_scores.masked.indel.hg38.vcf.gz
  * REVEL (version May 2022)
    * revel_b38.tsv.gz
  * CADD (v1.7)
    * cadd_1.7_b38_whole_genome_SNVs.tsv.gz
    * cadd.1.7.b38.gnomad.genomes.r4.0.indel.tsv.gz
  * Mastermind (version April 2022)
    * mastermind_cited_variants_reference-2022.04.02-grch38.vcf.gz



## Notes
  How to check the names of all the files included in the config:

```bash
config_file=your_file_name.json

# Get the Vep Resources filenames
for file in  $(jq -r ' .vep_resources | .[]' $config_file);
do dx describe $file --json | jq -r '.name';
done

# Get Custom Annotation filenames
for file in  $(jq -r ' .custom_annotations[]|.resource_files[]|.file_id' $config_file);
do dx describe $file --json | jq -r '.name';
done

# Get Plugin Annotation filenames
for file in  $(jq -r ' .plugins[]|.resource_files[]|.file_id' $config_file);
do dx describe $file --json | jq -r '.name';
done
```