# ALMetevsk

## Подключение
```{}
cd /home/alexandr/Downloads/063_annotator1
sudo openvpn externalwork3-client.conf
ssh oxkolpakova@pbx3
source activate alm
export PATH=$PATH:/home/oxkolpakova/programs/miniconda3/envs/alm/bin
scp -r /home/alexandr/Documents/ALM/data/raw/* oxkolpakova@pbx3:/home/oxkolpakova/data/raw
```
## Загрузка референса и создание индекса
```
datasets-cli download genome accession GCF_000001405.40 --include gff3,genome --filename GCF_000001405.40.zip

bwa mem index human.fna
```
## Качество
```
trimmomatic PE -threads 4 raw/202309251627_220601009_2P230329071US2S2721BX_B_neft250923_1_L00_R1.fq.gz raw/202309251627_220601009_2P230329071US2S2721BX_B_neft250923_1_L00_R2.fq.gz \
    results/trimmomatic/R1.qc.fq.gz results/trimmomatic/s1_se \
    results/trimmomatic/R2.qc.fq.gz results/trimmomatic/s2_se \
    ILLUMINACLIP:adapters.fa:2:40:15 \ 
    LEADING:2 TRAILING:2 \
    SLIDINGWINDOW:4:25 \ 
    MINLEN:50 
```


```
fastp -q 20 -l 50  --trim_poly_g --in1 raw/202309251627_220601009_2P230329071US2S2721BX_B_neft250923_1_L00_R1.fq.gz --in2 raw/202309251627_220601009_2P230329071US2S2721BX_B_neft250923_1_L00_R2.fq.gz \
  --out1 results/fastp/R1.qc.fq.gz --out2 results/trimmomatic/R2.qc.fq.gz \
#  --adapter_sequence adapters.fa \

```
## Выравнивание
Я хочу выравнить мои файлы с парными ридами на геном человека с помощью STAR, можешь написать команду?

Путь до референса fna:
/home/oxkolpakova/data/references/human.fna

Путь до файла разметки gtf:
/home/oxkolpakova/data/references/genomic.gtf

Путь до первого рида R1:
/home/oxkolpakova/data/202309251627_220601009_2P230329071US2S2721BX_B_neft250923_1_L00_R1.fq.gz

Путь до второго рида R2:
/home/oxkolpakova/data/202309251627_220601009_2P230329071US2S2721BX_B_neft250923_1_L00_R2.fq.gz

Путь до выходных файлов:
/home/oxkolpakova/data/result/human

Путь до невыравненных ридов:
/home/oxkolpakova/data/result/unmapped
```
bwa mem -t 12 /home/oxkolpakova/data/references/human.fna /home/oxkolpakova/data/202309251627_220601009_2P230329071US2S2721BX_B_neft250923_1_L00_R1.fq.gz /home/oxkolpakova/data/202309251627_220601009_2P230329071US2S2721BX_B_neft250923_1_L00_R2.fq.gz > /home/oxkolpakova/data/result/human/alignment.sam
```
# Сохраняем невыравненные
```
samtools view -Sb alignment.sam | samtools sort -o alignment.sorted.bam
samtools view -b -o /home/oxkolpakova/data/result/unmapped/unmapped.bam -f 4 /home/oxkolpakova/data/result/human/alignment.sam
samtools index samtools index alignment.sorted.bam 
samtools idxstats alignment.sorted.bam 
```
