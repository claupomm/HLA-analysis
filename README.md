# HLA analysis on RNA-seq/WES data
HLA (human leukocyte antigen) genes are encoded on chromosome 6 and are important for the immune system. Their highly polymorphic nature makes them suitable for cell line authentication and tissue compatibility. In the following HLA analysis on leukocyte/lymphocyte cell lines is outlined as available on DSMZCellDive (https://celldive.dsmz.de/hla) and published for a RNA-seq dataset here: https://f1000research.com/articles/11-420. 

Via arcasHLA RNA-seq as well as WES data can be analysed (Orenbuch et al, 2020, https://academic.oup.com/bioinformatics/article/36/1/33/5512361).

In the following HLA are analysed on RNA-seq basis.




## Analysis via arcasHLA
Before installing the HLA analysis tool, get required tools via conda:
```
DIR=/path/to/Project_folder
cd $DIR
export PATH="/your/path/to/anaconda3/bin:$PATH"
mamba create -n hla python=3.6  bedtools=2.27.1 kallisto=0.44.0
mamba activate hla
mamba install numpy scipy pandas pytest
mamba install -c conda-forge biopython=1.77
git clone https://github.com/RabadanLab/arcasHLA
cd arcasHLA
```
Test arcasHLA:
```
./arcasHLA extract test/test.bam
./arcasHLA genotype test/test.extracted.1.fq.gz test/test.extracted.2.fq.gz -g A,B,C,DPB1,DQB1,DQA1,DRB1-o test/output -t 7 -v
```
Run arcasHLA on your WES data
```
# run arcasHLA on WEES samples
DIR=/path/to/Project_folder/arcasHLA
cd $DIR

SAMPLES=$(ls /path/to/your/fastq_files/*R1_001.fastq.gz)
for SAMPLE in $SAMPLES # each sample <1min
do
sample=$(basename $SAMPLE)
sbatch -J $sample -o $sample.wes.out -e $sample.wes.err -c 7 -p ultrashort <<EOF
#!/bin/sh
./arcasHLA genotype $SAMPLE ${SAMPLE/_R1_001/_R2_001} -o ll100_wes/ -t 7
EOF
done
# merge several json files
./arcasHLA merge -i ll100_wes -o ll100_wes # saved in genotypes.csv
```
Run arcasHLA on your RNA-seq data
```
SAMPLES=$(ls /path/to/your/fastq_files/*_1.fastq.gz)
for SAMPLE in $SAMPLES # each sample <5min
do
sample=$(basename $SAMPLE)
sbatch -J $sample -o $sample.rna.out -e $sample.rna.err -c 7 -p ultrashort <<EOF
#!/bin/sh
./arcasHLA genotype $SAMPLE ${SAMPLE/_1./_2.} -o ll100_rna/ -t 7
EOF
done
# merge several json files
./arcasHLA merge -i ll100_rna -o ll100_rna # saved in genotypes.csv
```
