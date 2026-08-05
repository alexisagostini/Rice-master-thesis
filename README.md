# Rice-master-thesis

## Conda
```bash
conda install -c conda-forge ncbi-datasets-cli
```

## creation of a depository

```bash
mkdir -p rice_project/{metadata,raw,reference,scripts,results,logs,tmp}
cd rice_project
```
## recuperation of the informations
```bash
echo "https://academic.oup.com/gigascience/article/3/1/2047-217X-3-7/2682915" > metadata/3k_rice_papers.txt # paper Gigascience 2024
echo "https://www.nature.com/articles/s41586-018-0063-9" >> metadata/3k_rice_papers.txt # Paper Nature 2018
echo "https://snp-seek.irri.org/" >> metadata/3k_rice_papers.txt # SNP-seek
```
## Download of the reference's genome
```bash
datasets download genome accession GCF_001433935.1 --include genome
unzip ncbi_dataset.zip
cp ncbi_dataset/data/GCF_001433935.1/GCF_001433935.1_IRGSP-1.0_genomic.fna /data/alexis/rice_project/genome_ref_rice_NCBI.fasta
```
## Recuperation of metadata ENA
```bash
curl -L "https://www.ebi.ac.uk/ena/portal/api/filereport?accession=PRJEB6180&result=sample&fields=sample_accession,secondary_sample_accession,sample_alias,scientific_name,tax_id,description,collection_date,country,location,first_public,last_updated&format=tsv" \
-o metadata/PRJEB6180_ENA_samples.tsv
```
## Aggregation of ENA run-level metadata to sample-level metadata
```bash syntaxe helped by claude 
awk -F'\t' '
NR==1 {
  for (i=1; i<=NF; i++) col[$i]=i
  next
}
{
  sample=$col["sample_accession"]
  secondary=$col["secondary_sample_accession"]
  sci=$col["scientific_name"]
  platform=$col["instrument_platform"]
  model=$col["instrument_model"]

  n_runs[sample]++
  reads[sample]+=$col["read_count"]
  bases[sample]+=$col["base_count"]

  if (!(sample in sec)) sec[sample]=secondary
  if (!(sample in species)) species[sample]=sci
  if (!(sample in instr_platform)) instr_platform[sample]=platform
  if (!(sample in instr_model)) instr_model[sample]=model
}
END {
  OFS="\t"
  for (s in n_runs) {
    print s,sec[s],species[s],instr_platform[s],instr_model[s],n_runs[s],reads[s],bases[s]
  }
}
' PRJEB6180_ENA_runs.tsv \
} > PRJEB6180_sample_summary_from_runs.tsvoheader.tsvtific_name\tinstrument_platform\tinstrument_model\tn_runs\ttotal_reads\ttotal_bases"
```
## duplacte verification 
```bash
sort PRJEB6180_ENA_runs.tsv | uniq -d
cut -f1 PRJEB6180_ENA_runs.tsv | sort | uniq -c | sort -rn | awk '$1 > 1'
```
## Test with 10 samples
```bash
shuf -n 10 PRJEB6180_ENA_runs.tsv > test_10_sample_accessions.tsv
```
### Start the pipeline
```bash
cat << 'EOF' > run_pipeline_test.sh
NCBI_API_KEY= #your key
REF= "rice_project/genome_ref_rice_NCBI.fasta"
head $REF
SRA="rice_project/test_10_sample_accessions.tsv"
head $SRA
module load genomepanel-nf
nextflow run /data/alexis/genomepanel_nf/main.nf \
	-profile slurm -work-dir '/scratch/nf_tmp_alexis' \
	--outdir '/data/alexis/rice_project/metadata/test_10_individus' \
	--NCBI_API_key "$NCBI_API_KEY" \
	--reference $REF \
	--SRA_index $SRA \
	--ploidy 2 \
	--slurm_queue normal.168h
EOF
```
```bash
bash run_pipeline_test.sh
```
## the full pipeline
```bash
cat << 'EOF' > run_pipeline_test.sh
NCBI_API_KEY= #your key
REF= "rice_project/genome_ref_rice_NCBI.fasta"
head $REF
SRA="rice_project/PRJEB6180_ENA_runs.tsv"
head $SRA
module load genomepanel-nf
nextflow run /data/alexis/genomepanel_nf/main.nf \
	-profile slurm -work-dir '/scratch/nf_tmp_alexis' \
	--outdir '/data/alexis/rice_project/metadata/test_10_individus' \
	--NCBI_API_key "$NCBI_API_KEY" \
	--reference $REF \
	--SRA_index $SRA \
	--ploidy 2 \
	--slurm_queue normal.168h
```
