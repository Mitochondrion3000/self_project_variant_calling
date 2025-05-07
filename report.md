после того как мы скачали даныне нам нужно сделать все вот это 
---

## 1. Выравнивание reads с помощью `bwa mem`

Перед началом выравнивания необходимо проиндексировать референсный геном.

### 🔹 Шаг 1: Индексация референсного генома

```bash
bwa index /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna
```

#### Вывод после индексирования:

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

---

### 🔹 Шаг 2: Выравнивание чтений с помощью `bwa mem`

```bash
bwa mem -t 8 /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna read1.fastq.gz read2.fastq.gz | samtools view -Sb - > aligned.bam
```

* `-t 8` — использовать 8 потоков
* `samtools view -Sb -` — конвертация SAM в BAM и вывод в файл `aligned.bam`

---


---

### Инструменты для анализа соматических мутаций
1. **Mutect2 (GATK)**  
   - Популярный инструмент от Broad Institute.  
   - Использует Gradient Boosting (XGBoost-like) для фильтрации ложных вариантов.
     
```bash
#Mutect2
export OMP_NUM_THREADS={resources.cpus_per_task}
file_name=$(basename {input.paired[1]})
name=${{file_name%.sort*}}
gatk Mutect2 \
	--java-options -Xmx8G \
	--native-pair-hmm-threads 8 \
	--reference {input.ref} \
	-I {input.paired[0]} \
 	-I {input.paired[1]} \
 	-normal $name \
 	--germline-resource {input.gnomad} \
 	--genotype-germline-sites true \
 	--genotype-pon-sites true \
 	--interval-padding {params.interval_padding} \
 	-pon {input.pon} \
 	-L {input.target} \
 	--f1r2-tar-gz {output.f1r2} \
 	-O {output.vcf}
```

2. **Strelka**  
   - Высокая точность для SNV и инделов.  
   - Применяет HMM и ML-фильтры.
  
`     #Strelka
configureStrelkaSomaticWorkflow.py –exome \
	--normal={input.paired[1]} \
	--tumor={input.paired[0]} \
	--ref={input.ref} \
	--runDir=tmp \
	--callRegions {input.target}.gz`

3. **DeepSomatic**  
   - CNN + LSTM для предсказания мутаций.  
`#DeepSomatic
run_deepsomatic \
	--model_type=WGS \
	--ref={input.ref} \
	--reads_normal={input.paired[1]} \
	--reads_tumor={input.paired[0]} \
	--output_vcf=DeepSomatic.somatic.vcf.gz \
	--sample_name_tumor="{wildcards.seq_name}" \
	--sample_name_normal="{wildcards.normal}" \
	--num_shards=20 \
	--logging_dir=DeepSomatic \
	--intermediate_results_dir=DeepSomatic/tmp \
        --regions={input.target}`
4. **VarNet (Google Health)**  
   - Основан на сверточных сетях (CNN).  
`#VarNet
python /VarNet/filter.py \
	--sample_name {wildcards.seq_name} \
	--normal_bam {input.paired[1]} \
	--tumor_bam {input.paired[0]} \
	--processes 8 \
	--output_dir VarNet \
	--reference {input.ref} \
	--region_bed {input.target}
python /VarNet/predict.py \
	--sample_name {wildcards.seq_name} \
	--normal_bam {input.paired[1]} \
	--tumor_bam {input.paired[0]} \
	--processes 8 \
	--output_dir VarNet \
	--reference {input.ref}
`
5. **VarDict**  
   - Эффективен для гетерогенных опухолей.  
   - Использует Random Forest.
 `#VarDict
 VarDict
	–dedup \
	-th 8 \
	-G {input.ref} \
	-f 0.02 \
	-N {wildcards.seq_name} \
	-r 3 \
	-b "{input.paired[0]}|{input.paired[1]}" \
	-c 1 -S 2 -E 3 -g 4 {input.target} | testsomatic.R | var2vcf_paired.pl -N "{wildcards.seq_name}|{wildcards.normal}" -f 0.02 > {output}` 
     
