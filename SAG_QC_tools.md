# **SAGs QC: CheckM v2 and GUNC**

## Create a soft link to the main directory
```shell
ln -s /mnt/storage/ storage
```

## Install conda packages

### Install then activate mamba for installing QC tools
```shell
conda create -c conda-forge mamba -y --prefix storage/envs/mamba

source activate storage/envs/mamba
```

### Install the QC tools via Mamba
```shell
mamba create -c bioconda -c conda-forge checkm2 -y --prefix storage/envs/checkm2

mamba create -c bioconda gunc -y --prefix storage/envs/gunc
```

## Download reference database for each tool
```shell
source activate storage/envs/checkm2
checkm2 database --download

source activate storage/envs/gunc
gunc download_db /home/jupyter-tchang/databases
```

## Run GUNC and CheckM v2

### First, place SAGs (genome assembly) of interests into individual folders
```shell
cd ~/storage/data/daily-data/day2/uncurated_SAGs
mkdir -p genome_assemblies/AM-388 genome_assemblies/AM-390

for plate in AM-388 AM-390; do
    find ${plate}_WGS -maxdepth 2 -name "*_contigs.fasta" \
        -exec cp {} genome_assemblies/${plate} \; &
done
```

### Run GUNC

#### Activate conda environment
```shell
source activate storage/envs/gunc
```

#### Create output directory
```shell
mkdir gunc_output
```

#### Run GUNC for two plates at the same time using a for-loop
```shell
for plate in AM-388 AM-390; do
    gunc run \
        --input_dir genome_assemblies/${plate} \
        --file_suffix fasta \
        --out_dir gunc_output/${plate} \
        --use_species_level \
        --contig_taxonomy_output \
        --detailed_output \
        --db_file /home/jupyter-tchang/databases/gunc_db_progenomes2.1.dmnd \
        --threads 15 &
done
```

### Run CheckM2

#### Activate conda environment
```shell
source activate storage/envs/checkm2
```

#### Run CheckM2 for two plates at the same time using a for-loop
```shell
for plate in AM-388 AM-390; do
    checkm2 predict \
        -i genome_assemblies/${plate} \
        -x fasta \
        -o checkm2_output/${plate} \
        --database_path /home/jupyter-tchang/databases/CheckM2_database/uniref100.KO.1.dmnd \
        --threads 8 &
done
```