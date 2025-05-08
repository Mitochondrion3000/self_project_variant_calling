
```markdown
# Обработка данных NGS: выравнивание и удаление дубликатов

## 1. Индексация референсного генома

Перед выравниванием необходимо проиндексировать референсный геном (hg38):

```bash
bwa index /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna
```

**Вывод команды:**
```
[bwt_gen] Finished constructing BWT in 728 iterations.
[bwa_index] 2133.71 seconds elapse.
[bwa_index] Update BWT... 16.51 sec
[bwa_index] Pack forward-only FASTA... 17.75 sec
[bwa_index] Construct SA from BWT and Occ... 1369.41 sec
[main] Version: 0.7.17-r1188
[main] CMD: bwa index /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna
[main] Real time: 3993.403 sec; CPU: 3560.819 sec
```

## 2. Выравнивание чтений с помощью bwa mem

```bash
bwa mem -t 12 /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna \
/media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_1.fastq.gz \
/media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_2.fastq.gz \
| samtools view -b - > SRR7890879_aligned.bam \
&& { echo -e "\a"; notify-send "BWA Index" "Ура нафиг!"; }
```

**Параметры:**
- `-t 12` - использование 12 потоков
- `samtools view -b -` - конвертация SAM в BAM

**Вывод команды:**
```
[main] Version: 0.7.17-r1188 
[main] CMD: bwa mem -t 12 /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna /media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_1.fastq.gz /media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_2.fastq.gz 
[main] Real time: 1672.178 sec; CPU: 17157.415 sec
```

## 3. Удаление дубликатов с помощью Sambamba

Сначала создаем окружение для Sambamba:

```bash
conda create -n sambamba_env -c bioconda sambamba --y
conda activate sambamba_env --y
```

Затем выполняем удаление дубликатов:

```bash
sambamba markdup -t 12 SRR7890879_aligned.bam SRR7890879_aligned_deduplicated.bam
```

**Параметры:**
- `-t 12` - количество потоков
- `SRR7890879_aligned_deduplicated.bam` - выходной файл без дубликатов

**Вывод команды:**
```
finding positions of the duplicate reads in the file... 
sorted 36352820 end pairs and 38948 single ends (among them 0 unmatched pairs) 
collecting indices of duplicate reads... done in 3910 ms 
found 15783533 duplicates 
collected list of positions in 2 min 36 sec 
marking duplicates... 
total time elapsed: 7 min 55 sec
```

- Вызову вариантов
```
