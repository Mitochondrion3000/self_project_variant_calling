# Лог-отчёт: Построение пайплайна соматического вариант-коллинга

**Проект**: Анализ искусственного онкологического датасета (syn3)
**Исполнитель**: Иван
**Дата**: 2025-05-12
**Цель**: Построение и отладка пайплайна выявления соматических мутаций с использованием нескольких тулов: GATK Mutect2, LoFreq, Strelka2, FreeBayes.

---

## 1. Подготовка референса

* Разархивирован файл `GCF_000001405.40_GRCh38.p14_genomic.fna` — необходимо, т.к. GATK не работает с `.gz`
* Индексация:

  ```bash
  bwa index GCF_000001405.40_GRCh38.p14_genomic.fna
  samtools faidx GRCh37.fna
  gatk CreateSequenceDictionary -R GRCh37.fna -O GRCh37.dict
  ```

---

## 2. Выравнивание FASTQ

* **Normal (обычные):**

  ```bash
  bwa mem -t 12 GRCh37.fna normal_1.fq.gz normal_2.fq.gz | samtools view -Sb - > aligned_normal.bam
  ```
* **Tumor (опухолевые):**

  ```bash
  bwa mem -t 12 GRCh37.fna tumor_1.fq.gz tumor_2.fq.gz | samtools view -Sb - > aligned_tumor.bam
  ```
* Повторное выравнивание с GCF\_000001405.25\_GRCh37.p13:

  ```bash
  bwa mem -t 12 GCF_000001405.25_GRCh37.p13_genomic.fna.gz tumor_1.fq.gz tumor_2.fq.gz | samtools view -b - > aligned_tumor_g.bam
  ```

---

## 3. Обработка BAM

* **Сортировка:**

  ```bash
  samtools sort -@ 8 -o aligned_tumor_g_sorted.bam aligned_tumor_g.bam
  samtools sort -@ 8 -o aligned_normal_g_sorted.bam aligned_normal_g.bam
  ```
* **Добавление Read Groups:**

  ```bash
  gatk AddOrReplaceReadGroups -I aligned_tumor_g_sorted.bam -O tumor_with_rg.bam \
    --RGID tumor_group1 --RGLB lib1 --RGPL illumina --RGPU unit1 --RGSM tumor_sample

  gatk AddOrReplaceReadGroups -I aligned_normal_g_sorted.bam -O normal_with_rg.bam \
    --RGID normal_group1 --RGLB lib1 --RGPL illumina --RGPU unit1 --RGSM normal_sample
  ```
* **Индексация:**

  ```bash
  samtools index tumor_with_rg.bam
  samtools index normal_with_rg.bam
  ```

---

## 4. Проверка качества

* Проверка статистик:

  ```bash
  samtools flagstat aligned_tumor_g_sorted.bam
  ```
* Проверка наличия тегов `@RG`:

  ```bash
  samtools view -H tumor_with_rg.bam | grep '^@RG'
  ```

---

## 5. GATK Mutect2

````bash

```bash
bcftools view -v snps somatic.vcf > somatic_snvs.vcf
bcftools view -v indels somatic.vcf > somatic_indels.vcf
```bash
gatk Mutect2 \
  -R hg19.fa \
  -I tumor_with_rg.bam \
  -I normal_with_rg.bam \
  -normal normal_sample \
  -L NGv3_fixed.bed \
  --f1r2-tar-gz f1r2.tar.gz \
  -O somatic.vcf.gz
````

---

## 6. LoFreq

* **Добавление indel-качества:**

  ```bash
  lofreq indelqual --dindel --ref hg19.fa -o tumor.dindel.bam tumor_with_rg.bam
  lofreq indelqual --dindel --ref hg19.fa -o normal.dindel.bam normal_with_rg.bam
  ```
* **Индексация BAM-файлов с индел-качеством:**

  ```bash
  samtools index tumor.dindel.bam
  samtools index normal.dindel.bam
  ```
* **Соматический вызов:**

  ```bash
  lofreq somatic \
    --call-indels \
    -n normal.dindel.bam \
    -t tumor.dindel.bam \
    -f hg19.fa \
    --threads 12 \
    -l NGv3_fixed.bed \
    -o sample1_vs_control1.LoFreq.vcf
  ```

---

## 7. Strelka2

* **Подготовка BED:**

  ```bash
  sort -k1,1 -k2,2n NGv3_fixed.bed > NGv3_fixed.sorted.bed
  bgzip NGv3_fixed.sorted.bed
  tabix -p bed NGv3_fixed.sorted.bed.gz
  ```

* **Запуск:**

  ```bash
  configureStrelkaSomaticWorkflow.py --exome \
    --normal normal_with_rg.bam \
    --tumor tumor_with_rg.bam \
    --ref hg19.fa \
    --runDir strelka_analysis \
    --callRegions NGv3_fixed.sorted.bed.gz

  cd strelka_analysis
  ./runWorkflow.py -m local -j 2 --memGb=10
  ```

---

## 8. FreeBayes

```bash
freebayes \
  -b normal_with_rg.bam \
  -b tumor_with_rg.bam \
  -t NGv3_fixed.bed \
  -f hg19.fa \
  -F 0.02 --min-coverage 10 -C 2 -m 30 -q 20 \
  --pooled-discrete --pooled-continuous --allele-balance-priors-off > output_freebayes.vcf
```

---

---

## 10. Возникшие сбои и их устранение

| № | Проблема                            | Причина                           | Решение                                          |
| - | ----------------------------------- | --------------------------------- | ------------------------------------------------ |
| 1 | Частые сбои пайплайна               | Отсутствие автоматизации          | Сохранение всех шагов, переход на WDL/Snakemake  |
| 2 | GATK не работает с `.fna.gz`        | Требуется разархивированный FASTA | Использовать `gunzip` и пересоздать индексы      |
| 3 | Несовпадение хромосом в BED         | `chr1` vs `1`                     | `sed 's/^/chr/' NGv3.bed > NGv3_fixed.bed`       |
| 4 | NCBI-референс не скачивался         | FTP нестабилен                    | Использовать UCSC зеркало                        |
| 5 | Файлы normal vs normal\_g           | Неясное происхождение             | Сравнение времени и хэшей файлов                 |
| 6 | Отсутствие Read Groups              | Не запускаются тулзы              | Добавлены через GATK `AddOrReplaceReadGroups`    |
| 7 | DeepSomatic не работает             | Не указаны ошибки                 | Проверка окружения, зависимостей, сборка вручную |
| 8 | Strelka требует индексированный BED | Формат ошибки                     | `sort + bgzip + tabix`                           |
| 9 | bcbio не установился                | Недостаточно ресурсов             | Использовать FreeBayes вручную                   |

---

## Вывод

Несмотря на множественные технические проблемы и несовместимости, пайплайн был успешно реализован. Использованы 4 независимых тулзы, BAM-файлы корректно подготовлены, получены VCF с SNV и indel. Лог документирует критические шаги, ошибки и пути их устранения.

Документ можно использовать как основу для публикации, отчёта по проекту или внутреннего аудита NGS-процедур.
