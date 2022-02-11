# myelofibrosis_project

# PROJECT: SOMATIC VARIANTS IN MYELOFIBROSIS
The objective of the project is to identify and demonstrate somatic variants related to the development of different myelofibrosis conditions in a group of adult patients, producing data that can help in the investigation and decision-making process in clinical practice.


## Team
#### Students
[Ariane Vieira](https://github.com/Arivsouza)
[Bruno Marcarini](https://github.com/Brunomarcarini)
[Daniela Varela](https://github.com/danielavarela)
[Marcia Nagamine](https://github.com/MarciaNagamine)

#### Academic advisor
[Renato Puga](https://github.com/renatopuga)

## Files

-  30 files in vcf format
- Excell table with information about sex, age, extracted material ..

####Creating the somatic and reference directories
```bash
mkdir /mnt/c/somatico
mkdir /mnt/c/somatico/vcfs
mkdir /mnt/c/somatico/annotation
mkdir /mnt/c/reference
```
#### Unzip the hg19.fa.gz file
```bash
cd /mnt/c/reference
wget -c https://hgdownload.soe.ucsc.edu/goldenPath/hg19/chromosomes/hg19.fa.gz
gunzip hg19.fa.gz
```
#### Vep installation

```bash
sudo apt install unzip curl git libmodule-build-perl libdbi-perl libdbd-mysql-perl build-essential zlib1g-dev
git clone https://github.com/Ensembl/ensembl-vep.git
cd ensembl-vep
./INSTALL.pl
```
##  Running the Vep
##### https://www.ensembl.org/info/docs/tools/vep/script/vep_options.html#basic

```bash
for file in/mnt/c/somatico/vcfs/*
do
/home/user/ensembl-vep/vep \
	-i "$file"\
	-o "$file".tabela.vcf \
    --cache --offline --assembly GRCh37 --refseq \
    --dir_cache /mnt/c/reference/VEP \ 
	--pick --pick_allele --force_overwrite --tab --symbol --check_existing \
    --everything \
	--fasta /mnt/c/reference/hg19.fa \
	--individual all 
    done
```
#### Installing bctools and the plugin

```bash
git clone --recurse-submodules git://github.com/samtools/htslib.git
git clone git://github.com/samtools/bcftools.git
cd bcftools
export BCFTOOLS_PLUGINS=/path/to/bcftools/plugins
 ### The following is optional:
 #### autoheader && autoconf && ./configure --enable-libgsl --enable-perl-filters
make
```
##### - Creating the header
```bash
for file in /mnt/c/somatico/annotation/*.tabela.vcf
do
bcftools +split-vep -l "$file" | cut -f2 | tr '\n\r' '\t' | awk '{print("CHROM\tPOS\tREF\t"$0"FILTER\tGT\tAD\tAF\tDP\tF1R2\tF2R1\tSB")}' > "$file".vep.tsv
done
```
##### - Adding all information and columns
```bash
for file in /mnt/c/somatico/annotation/*.tabela.vcf
do
bcftools +split-vep -f '%CHROM\t%POS\t%REF\t%CSQ\t%FILTER\t[%GT\t%AD\t%AF\t%DP\t%F1R2\t%F2R1\t%SB]\n' -d -A tab "$file" >> "$file".vep.tsv
done
```
#### Concatenate the generated files
```bash
cat*.tsv>todos_vep.tsv
```
