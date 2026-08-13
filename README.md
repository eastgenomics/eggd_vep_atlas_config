# eggd_vep_atlas_config
JSON configuration file for VEP in the atlas pipeline

## What does the JSON config do?

Configuration file required to annotate a vcf using [Variant Effect Predictor](https://github.com/Ensembl/ensembl-vep) implementation [eggd_vep](https://github.com/eastgenomics/eggd_vep).

A variable level of annotation can be achieved by different combinations of custom annotations and vep plugins, in addition to the required VEP resources.

## What does the JSON config version contain?

This json file provides information about annotations,plugins, required fields and the genome version.

* Genome build: GRCh38
* VEP required files:
  * vep_v115.2.tar.gz
  * homo_sapiens_refseq_vep_115_GRCh38.tar.gz
  * plugin_config.txt
  * Homo_sapiens_vep_115.GRCh38.dna.toplevel.fa.gz
  * Homo_sapiens_vep_115.GRCh38.dna.toplevel.fa.gz.fai
  * Homo_sapiens_vep_115.GRCh38.dna.toplevel.fa.gz.gzi
  * GRCh38_GIABv3_no_alt_analysis_set_maskedGRC_decoys_MAP2K3_KMT2C_KCNJ18_noChr.fasta-index.tar.gz
* Custom Annotation sources:
  * clinvar_20260510_GRCh38.vcf.gz
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

# All in one command
jq -r '
  def ids($tag): to_entries[]
    | select(.value | type == "string" and startswith("file-"))
    | [$tag, .key, .value];
  (.vep_resources                      | ids("vep_resources")),
  (.custom_annotations[] | .name as $n | .resource_files[] | ids("custom:\($n)")),
  (.plugins[]            | .name as $n | ids("plugin:\($n)"),
                                         (.resource_files[] | ids("plugin:\($n)")))
  | @tsv' "$config_file" |
while IFS=$'\t' read -r section field file_id; do
    name=$(dx describe --name "$file_id" 2>/dev/null) || name="<<UNRESOLVED>>"
    printf '%s\t%s\t%s\t%s\n' "$section" "$field" "$file_id" "${name:-<<UNRESOLVED>>}"
done | column -t -s$'\t'
```