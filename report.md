### Загрузка и обработка данных

#### 1. Скачивание сырых прочтений (FASTQ)
Сырые данные (raw reads) доступны в NCBI SRA:  
🔗 [SRR7890879](https://trace.ncbi.nlm.nih.gov/Traces/?view=run_browser&acc=SRR7890879&display=data-access)  

**Примечание**: Разница между форматами SRA: 
![image](https://github.com/user-attachments/assets/45a94084-ae0d-48ea-bfa2-cdfbd8dfe9f4)

- **SRA Lite** – облегченная версия без некоторых метаданных.  
- **SRA Normalized** – данные нормализованы (например, по глубине покрытия).  

Для загрузки FASTQ напрямую (без промежуточного .sra) использовалась команда:  
```bash
fasterq-dump --split-files --progress --threads 4 SRR7890883
```

#### 2. Загрузка датасета cancer-dream-syn3
```bash
mkdir -p cancer-dream-syn3/config cancer-dream-syn3/input cancer-dream-syn3/work
cd cancer-dream-syn3/config
wget https://raw.githubusercontent.com/bcbio/bcbio-nextgen/master/config/examples/cancer-dream-syn3.yaml
cd ../input
wget https://raw.githubusercontent.com/bcbio/bcbio-nextgen/master/config/examples/cancer-dream-syn3-getdata.sh
bash cancer-dream-syn3-getdata.sh
```

#### 3. Референсный геном
Скачан GRCh38 (GCF_000001405.40) из NCBI Datasets, так как он указан в референсных VCF:  
```bash
conda activate ncbi_datasets
datasets download genome accession GCF_000001405.40 --include genome
```
Для экономии места можно использовать `--dehydrated` с последующей регидратацией:  
```bash
datasets rehydrate --directory ncbi_dataset
```

#### 4. Конвертация SRA Lite в FASTQ
```bash
conda activate sra_env
fastq-dump --split-files --gzip /path/to/SRR7890879.sralite.1
```
**Результат**: 36,393,620 прочтений.

#### 5. Выравнивание на референс
Использован `bwa mem` для выравнивания парных прочтений:  
```bash
bwa mem -t 8 /path/to/hg38.fasta sample_1.fastq.gz sample_2.fastq.gz | samtools view -Sb - > aligned.bam
```

---

### Инструменты для анализа соматических мутаций
1. **Mutect2 (GATK)**  
   - Популярный инструмент от Broad Institute.  
   - Использует Gradient Boosting (XGBoost-like) для фильтрации ложных вариантов.
     
`#Mutect2
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
 	-O {output.vcf}`

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
     
