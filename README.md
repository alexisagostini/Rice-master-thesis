# Rice-master-thesis

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
' metadata/PRJEB6180_ENA_runs.tsv \
} > metadata/PRJEB6180_sample_summary_from_runs.tsvoheader.tsvtific_name\tinstrument_platform\tinstrument_model\tn_runs\ttotal_reads\ttotal_bases"
```
## Test with 10 sample
```bash
awk -F'\t' '
NR==1 {
  for (i=1; i<=NF; i++) col[$i]=i
  print "sample_id,run_accession"
  next
}
{
  sample=$col["sample_accession"]
  run=$col["run_accession"]
  layout=$col["library_layout"]
  fastq=$col["fastq_ftp"]

  if (layout=="PAIRED" && fastq!="") {
    if (!(sample in seen)) {
      seen[sample]=1
      n++
      print "rice_"n","run
    }
  }

  if (n==10) exit
}
' metadata/PRJEB6180_ENA_runs.tsv > metadata/test_10_sra_index.csv
```
## create a descriptive table
```bash
awk -F'\t' '
NR==1 {
  for (i=1; i<=NF; i++) col[$i]=i
  print "sample_id\tsample_accession\tsecondary_sample_accession\trun_accession\tread_count\tbase_count\tinstrument_model"
  next
}
{
  sample=$col["sample_accession"]
  secondary=$col["secondary_sample_accession"]
  run=$col["run_accession"]
  reads=$col["read_count"]
  bases=$col["base_count"]
  model=$col["instrument_model"]
  layout=$col["library_layout"]
  fastq=$col["fastq_ftp"]

  if (layout=="PAIRED" && fastq!="") {
    if (!(sample in seen)) {
      seen[sample]=1
      n++
      print "rice_"n"\t"sample"\t"secondary"\t"run"\t"reads"\t"bases"\t"model
    }
  }

  if (n==10) exit
}
' metadata/PRJEB6180_ENA_runs.tsv > metadata/test_10_samples_description.tsv
```
