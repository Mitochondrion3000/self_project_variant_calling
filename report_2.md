Работаем с искусственным датасетом 

1) Выравниваем на рефернес вначале нормальные
   bwa mem -t 12 /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference/GRCh37.fna /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_normal_NGv3_1.fq.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_normal_NGv3_2.fq.gz | samtools view -Sb - > aligned_normal.bam
bwa mem -t 12 /media/ivan/KINGSTON/self_project/cancer-dream-syn3/reference/GRCh37.fna /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_tumor_NGv3_1.fq.gz /media/ivan/KINGSTON/self_project/cancer-dream-syn3/input/synthetic_challenge_set3_tumor_NGv3_2.fq.gz | samtools view -Sb - > aligned.bam
