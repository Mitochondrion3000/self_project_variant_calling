bwa index /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna
samtools faidx /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference/GRCh37.fna для референса не нужно эту тему забывать 
gatk CreateSequenceDictionary \
  -R /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference/GRCh37.fna \
  -O /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference/GRCh37.dict и вот эту 

Работаем с искусственным датасетом 

1) Выравниваем на рефернес вначале нормальные
   bwa mem -t 12 /media/ivan/KINGSTON/GWAS/reference_data/GRCh37.fna /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_normal_NGv3_1.fq.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_normal_NGv3_2.fq.gz | samtools view -Sb - > aligned.bam
   [main] CMD: bwa mem -t 12 /media/ivan/KINGSTON/GWAS/reference_data/GRCh37.fna /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_normal_NGv3_1.fq.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_normal_NGv3_2.fq.gz
[main] Real time: 1583.621 sec; CPU: 16009.305 sec

bwa mem -t 12 /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference/GRCh37.fna /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_tumor_NGv3_1.fq.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_tumor_NGv3_2.fq.gz | samtools view -b - > aligned_tumor.bam
[main] CMD: bwa mem -t 12 /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference/GRCh37.fna /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_tumor_NGv3_1.fq.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_tumor_NGv3_2.fq.gz
[main] Real time: 1547.319 sec; CPU: 15858.617 sec







bwa index /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/GCF_000001405.25_GRCh37.p13_genomic.fna.gz
samtools /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/GCF_000001405.25_GRCh37.p13_genomic.fna.gz 
gatk CreateSequenceDictionary \
  -R /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/GCF_000001405.25_GRCh37.p13_genomic.fna.gz \
  -O /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference/GRCh37.dict
bwa mem -t 12 /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/GCF_000001405.25_GRCh37.p13_genomic.fna.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_tumor_NGv3_1.fq.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_tumor_NGv3_2.fq.gz | samtools view -b - > aligned_tumor_g.bam
bwa mem -t 12  /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/GCF_000001405.25_GRCh37.p13_genomic.fna.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_normal_NGv3_1.fq.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_normal_NGv3_2.fq.gz | samtools view -Sb - > aligned_normal_g.bam


я задолбался сколько раз у меня все слетало 

gatk --java-options "-Xmx8G -DGATK_STACKTRACE_ON_USER_EXCEPTION=true"   Mutect2   --native-pair-hmm-threads 8   --reference "/media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/hg19.fa"   -I "/media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/tumor_with_rg.bam"   -I "/media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/normal_with_rg.bam"   -normal "normal_sample"   --interval-padding 50   -L "/media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/NGv3_fixed.bed"   --f1r2-tar-gz "/media/ivan/KINGSTON/self_project/cancer-dream-syn3/results/f1r2.tar.gz"   -O "/media/ivan/KINGSTON/self_project/cancer-dream-syn3/results/somatic.vcf.gz" вот такая вот рабочая команда вышла по итогу 
для того чтобы она заработало нужно было 

С референсом: 
1) Разархивировать, с арфивными файлами эта муть не работает
2) bwa index /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna
samtools faidx /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference/GRCh37.fna для референса не нужно эту тему забывать 
gatk CreateSequenceDictionary \
  -R /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference/GRCh37.fna \
  -O /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference/GRCh37.dict и вот эту и вот такие штуки для референсов нужно было сделать
Опухолевый файл: 
samtools index /media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/aligned_tumor_g.bam
samtools sort -@ 8 \
  -o /полный/путь/aligned_tumor_g_sorted.bam \
  /media/ivan/KINGSTON/.../aligned_tumor.bam
Обычный файл:

sed 's/^/chr/' /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/NGv3.bed > NGv3_fixed.bed
нужно хромосомы исправить

что еще важно брал я референсный датасет(https://gatk.broadinstitute.org/hc/en-us/articles/360035890711-GRCh37-hg19-b37-humanG1Kv37-Human-Reference-Discrepancies#grch37) там где хромосомы так называются chrX
вот тут (http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/hg19.fa.gz), а вот этот почему-то нормально не скачивался(ftp://ftp.ncbi.nlm.nih.gov/genomes/refseq/vertebrate_mammalian/Homo_sapiens/all_assembly_versions/GCF_000001405.25_GRCh37.p13/GCF_000001405.25_GRCh37.p13_genomic.fna.gz) 
__
Расписываем все что мы должы были делать с BAM файлами
и тут проблема я не помню почему у меня есть файлы с normal_g и просто normal я беру normal_g просто потому что они свежее
1) Сортируем оба файла
   samtools sort -@ 8 -o aligned_tumor_g_sorted.bam aligned_tumor_g.bam
   samtools sort -@ 8 -o aligned_normal_g_sorted.bam aligned_normal_g.bam
2) Добавляем тэги, это по видимому очень важно
   
   gatk AddOrReplaceReadGroups \
    -I aligned_tumor_g_sorted.bam \
    -O tumor_with_rg.bam \
    --RGID "tumor_group1" \
    --RGLB "lib1" \
    --RGPL "illumina" \
    --RGPU "unit1" \
    --RGSM "tumor_sample"
   
   gatk AddOrReplaceReadGroups \
    -I aligned_normal_g_sorted.bam \
    -O normal_with_rg.bam \
    --RGID "normal_group1" \          # Должен отличаться от tumor_group1!
    --RGLB "lib1" \                  # Можно оставить как в tumor или изменить
    --RGPL "illumina" \              # Должен совпадать с tumor (если секвенирование на одной платформе)
    --RGPU "unit1" \                 # Можно оставить как в tumor или изменить
    --RGSM "normal_sample"           # Должен отличаться от tumor_sample!

3) Проверяем заголовки
   samtools view -H normal_with_rg.bam | grep '@RG'
   samtools view -H tumor_with_rg.bam | grep '@RG'
5) Индесируем
   samtools index normal_with_rg.bam
   samtools index tumor_with_rg.bam

   

__

вот так вот проверка качества  BAM файлов делается 
samtools flagstat aligned_tumor_g_sorted.bam
97898571 + 0 in total (QC-passed reads + QC-failed reads)
97675052 + 0 primary
0 + 0 secondary
223519 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
96413067 + 0 mapped (98.48% : N/A)
96189548 + 0 primary mapped (98.48% : N/A)
97675052 + 0 paired in sequencing
48837526 + 0 read1
48837526 + 0 read2
84016350 + 0 properly paired (86.02% : N/A)
94712998 + 0 with itself and mate mapped
1476550 + 0 singletons (1.51% : N/A)
1669170 + 0 with mate mapped to a different chr
1087699 + 0 with mate mapped to a different chr (mapQ>=5)

проверка наличия тегов 
samtools view -H tumor_with_rg.bam | grep '^@RG'

(base) ivan@ivan-Redmi-Book-Pro-15-2022:/media/ivan/KINGSTON/self_project/cancer-dream-syn3/work$ samtools view -H aligned_tumor_g_sorted.bam | grep '^@RG'
(base) ivan@ivan-Redmi-Book-Pro-15-2022:/media/ivan/KINGSTON/self_project/cancer-dream-syn3/work$ samtools view -H aligned_normal_g_sorted.bam | grep '^@RG'
@RG	ID:group1	LB:lib1	PL:illumina	SM:normal_sample	PU:unit1
(base) ivan@ivan-Redmi-Book-Pro-15-2022:/media/ivan/KINGSTON/self_project/cancer

для опузоли добавим тэги 
gatk AddOrReplaceReadGroups \
    -I aligned_tumor_g_sorted.bam \
    -O tumor_with_rg.bam \
    --RGID "tumor_group1" \
    --RGLB "lib1" \
    --RGPL "illumina" \
    --RGPU "unit1" \
    --RGSM "tumor_sample" \
    --RGCN "sequencing_center"

а это мы для нормального добовляли 
picard AddOrReplaceReadGroups     I=aligned_normal_g.bam     O=aligned_normal_g_fixed.bam     RGID=group1     RGLB=lib1     RGPL=illumina     RGPU=unit1     RGSM=normal_sample  # Здесь задайте осмысленное имя образца и это имя потом передать нужно -normal "normal_sample" сюда
файл с интересующими нас участками

bcftools view -v snps /media/ivan/KINGSTON/self_project/cancer-dream-syn3/results/somatic.vcf > somatic_snvs.vcf
bcftools view -v indels /media/ivan/KINGSTON/self_project/cancer-dream-syn3/results/somatic.vcf > somatic_indels.vcf
___ следующий надеюсь рабочий

conda create -n lofreq_env -c bioconda lofreq
conda activate lofreq_env
lofreq version


# Шаг 1: Добавление индел-качества в нормальный BAM
lofreq indelqual --dindel \
    --ref /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/hg19.fa \
    -o normal.dindel.bam \
    /media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/normal_with_rg.bam

samtools index /media/ivan/KINGSTON/self_project/cancer-dream-syn3/results/normal.dindel.bam
samtools index /media/ivan/KINGSTON/self_project/cancer-dream-syn3/results/tumor.dindel.bam

# Шаг 2: То же самое для опухолевого
lofreq indelqual --dindel \
    --ref /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/hg19.fa \
    -o tumor.dindel.bam \
    /media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/tumor_with_rg.bam

# Шаг 3: Сравнение нормального и опухолевого (теперь используем *_dindel.bam!)
lofreq somatic --call-indels \
    -n normal.dindel.bam \
    -t tumor.dindel.bam \
    -f /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/hg19.fa \
    --threads 12 \
    -l /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/NGv3_fixed.bed \
    -o sample1_vs_control1.LoFreq.





___
Тепрь следующий инструмент:
Тут почему-то запуск делется на файлы конфигурации
configureStrelkaSomaticWorkflow.py --exome \
    --normal=normal_sample.bam \
    --tumor=tumor_sample.bam \
    --ref=hg38.fa \
    --runDir=strelka_analysis \
    --callRegions target_regions.bed.gz

команда с прописаными путями:

conda activate strelka_env

configureStrelkaSomaticWorkflow.py --exome \
    --normal=/media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/normal_with_rg.bam \
    --tumor=/media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/tumor_with_rg.bam \
    --ref=/media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/hg19.fa \
    --runDir=/media/ivan/KINGSTON/self_project/cancer-dream-syn3/results/strelka_analysis \
    --callRegions=/media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/NGv3_fixed.bed


А стрелка требует манипуляции с bed файлом которые не требовал gatk

# Переходим в директорию с BED-файлом
cd /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/

# Сортируем BED (обязательно!)
sort -k1,1 -k2,2n NGv3_fixed.bed > NGv3_fixed.sorted.bed

# Сжимаем с помощью bgzip
bgzip NGv3_fixed.sorted.bed

# Создаём индекс .tbi
tabix -p bed NGv3_fixed.sorted.bed.gz

Successfully created workflow run script.
To execute the workflow, run the following script and set appropriate options:

/media/ivan/KINGSTON/self_project/cancer-dream-syn3/results/strelka_analysis/runWorkflow.py

# 1. Освободите память (закройте тяжёлые приложения)
# 2. Запустите Strelka с ограничениями
cd /media/ivan/KINGSTON/self_project/cancer-dream-syn3/results/
./strelka_analysis/runWorkflow.py -m local -j 2 --memGb=10
__

Еще один инструмент deepsomatic

conda create -n deepsomatic_env -y
conda activate deepsomatic_env

conda install pytorch torchvision torchaudio cpuonly -c pytorch
pip install numpy pysam cython

git clone https://github.com/bioinformatics-centre/DeepSomatic.git
cd DeepSomatic


git clone https://github.com/bioinformatics-centre/DeepSomatic.git
cd DeepSomatic он тоже не пошел не знаю почему 
___

еще один freebayes

conda create -n freebayes_env
conda activate freebayes_env

conda install -c bioconda freebayes

freebayes \
    -b /media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/normal_with_rg.bam \
    -b /media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/tumor_with_rg.bam \
    -t /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/NGv3_fixed.bed \
    -f /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/hg19.fa \
    -F 0.02 \
    --min-coverage 10 \
    -C 2 \
    -m 30 \
    -q 20 \
    -R 0 \
    -S 0 \
    --pooled-discrete \
    --pooled-continuous \
    --allele-balance-priors-off > output_freebayes.vcf
вот эта команда вроде успешно сработала 

#Additional script see https://github.com/bcbio/bcbio-nextgen/blob/master/bcbio/variation/freebayes.py#L118
    
ivan@ivan-Redmi-Book-Pro-15-2022:/media/ivan/KINGSTON/self_project$ samtools view -H /media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/normal_with_rg.bam | grep '@RG'
@RG	ID:normal_group1	LB:lib1	PL:illumina	SM:normal_sample	PU:unit1
(freebayes_env) ivan@ivan-Redmi-Book-Pro-15-2022:/media/ivan/KINGSTON/self_project$ samtools view -H /media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/tumor_with_rg.bam | grep '@RG'
@RG	ID:tumor_group1	LB:lib1	PL:illumina	SM:tumor_sample	PU:unit1

conda install -c bioconda freebayes six toolz bcftools vcflib vt
------------------------python bcbio_nextgen_install.py /home/ivan/bcbio --tooldir=~/bcbio/tools --nodata не хватило ресуров чтобы уставновить 

python /media/ivan/KINGSTON/self_project/freebayes.py     /media/ivan/KINGSTON/self_project/output_freebayes.vcf     tumor_sample     normal_sample     > /media/ivan/KINGSTON/self_project/final.vcf



