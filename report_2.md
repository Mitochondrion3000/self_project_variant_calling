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



picard AddOrReplaceReadGroups     I=aligned_normal_g.bam     O=aligned_normal_g_fixed.bam     RGID=group1     RGLB=lib1     RGPL=illumina     RGPU=unit1     RGSM=normal_sample  # Здесь задайте осмысленное имя образца


bwa index /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/GCF_000001405.25_GRCh37.p13_genomic.fna.gz
samtools /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/GCF_000001405.25_GRCh37.p13_genomic.fna.gz 
gatk CreateSequenceDictionary \
  -R /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/GCF_000001405.25_GRCh37.p13_genomic.fna.gz \
  -O /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference/GRCh37.dict
bwa mem -t 12 /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/GCF_000001405.25_GRCh37.p13_genomic.fna.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_tumor_NGv3_1.fq.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_tumor_NGv3_2.fq.gz | samtools view -b - > aligned_tumor_g.bam
bwa mem -t 12  /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference_gatk/GCF_000001405.25_GRCh37.p13_genomic.fna.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_normal_NGv3_1.fq.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_normal_NGv3_2.fq.gz | samtools view -Sb - > aligned_normal_g.bam

  
