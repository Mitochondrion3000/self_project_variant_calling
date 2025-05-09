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

gatk --java-options "-Xmx8G -DGATK_STACKTRACE_ON_USER_EXCEPTION=true"   Mutect2   --native-pair-hmm-threads 8   --reference "/media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/hg19.fa"   -I "/media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/aligned_tumor_g_sorted.bam"   -I "/media/ivan/KINGSTON/self_project/cancer-dream-syn3/work/aligned_normal_g_sorted.bam"   -normal "normal_sample"   --interval-padding 50   -L "/media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/NGv3_fixed.bed"   --f1r2-tar-gz "/media/ivan/KINGSTON/self_project/cancer-dream-syn3/results/f1r2.tar.gz"   -O "/media/ivan/KINGSTON/self_project/cancer-dream-syn3/results" вот такая вот рабочая команда вышла по итогу 
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
picard AddOrReplaceReadGroups     I=aligned_normal_g.bam     O=aligned_normal_g_fixed.bam     RGID=group1     RGLB=lib1     RGPL=illumina     RGPU=unit1     RGSM=normal_sample  # Здесь задайте осмысленное имя образца и это имя потом передать нужно -normal "normal_sample" сюда
файл с интересующими нас участками
sed 's/^/chr/' /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/NGv3.bed > NGv3_fixed.bed
нужно хромосомы исправить



  
