после того как мы скачали даныне нам нужно сделать все вот это: 

The raw reads were aligned using bwa mem to the human genome reference (hg19 or hg38) corresponding to the reference used for the true set of somatic variants for each dataset [26]. Duplicated reads were marked with sambamba, and base quality score recalibration (BQSR) was done with GATK4 [27, 28]. Influence of post-alignment procedures on variant calling was evaluated using four different strategies (bwa; bwa + deduplication; bwa + BQSR; bwa + deduplication + BQSR).
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
реальная команда 
(base) ivan@ivan-Redmi-Book-Pro-15-2022:/media/ivan/KINGSTON/self_project/bams$ bwa mem -t 12 /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna /media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_1.fastq.gz /media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_2.fastq.gz | samtools view -b - > SRR7890879_aligned.bam && { echo -e "\a"; notify-send "BWA Index" "Ура нафиг!"; }

вывод команды
(base) ivan@ivan-Redmi-Book-Pro-15-2022:/media/ivan/KINGSTON/self_project/bams$ bwa mem -t 12 /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna /media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_1.fastq.gz /media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_2.fastq.gz | samtools view -b - > SRR7890879_aligned.bam && { echo -e "\a"; notify-send "BWA Index" "Ура нафиг!"; }


[main] Version: 0.7.17-r1188
[main] CMD: bwa mem -t 12 /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna /media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_1.fastq.gz /media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_2.fastq.gz
[main] Real time: 1672.178 sec; CPU: 17157.415 sec

* `-t 8` — использовать 8 потоков
* `samtools view -Sb -` — конвертация SAM в BAM и вывод в файл `aligned.bam`

2. Удаление дубликатов с помощью sambamba markdup
bash
conda create -n sambamba_env -c bioconda sambamba --y
conda activate sambamba_env
--y автоматический ответ да на запросы

sambamba markdup -t 12 SRR7890879_aligned.bam SRR7890879_aligned_deduplicated.bam
реальная команда sambamba markdup -t 12 SRR7890879_aligned.bam SRR7890879_aligned_deduplicated.bam
Где:

    -t 8 — количество потоков.

    deduplicated.bam — выходной файл без дубликатов.

вывод команды 
finding positions of the duplicate reads in the file...
  sorted 36352820 end pairs
     and 38948 single ends (among them 0 unmatched pairs)
  collecting indices of duplicate reads...   done in 3910 ms
  found 15783533 duplicates
collected list of positions in 2 min 36 sec
marking duplicates...
total time elapsed: 7 min 55 sec

4. Базовоевая рекалибровка качества (BQSR) с помощью GATK4
   перед вот этим всем нужно еще вот какую штуку
   samtools faidx /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna
команда которая не выдает никаких логов

и еще вот эту шнягу замутить 
gatk CreateSequenceDictionary \
    -R /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna \
    -O /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.dict

вывод команды 

Using GATK jar /home/ivan/anaconda3/envs/gatk_env/share/gatk4-4.3.0.0-0/gatk-package-4.3.0.0-local.jar
Running:
    java -Dsamjdk.use_async_io_read_samtools=false -Dsamjdk.use_async_io_write_samtools=true -Dsamjdk.use_async_io_write_tribble=false -Dsamjdk.compression_level=2 -jar /home/ivan/anaconda3/envs/gatk_env/share/gatk4-4.3.0.0-0/gatk-package-4.3.0.0-local.jar CreateSequenceDictionary -R /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna -O /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.dict
21:13:17.201 INFO  NativeLibraryLoader - Loading libgkl_compression.so from jar:file:/home/ivan/anaconda3/envs/gatk_env/share/gatk4-4.3.0.0-0/gatk-package-4.3.0.0-local.jar!/com/intel/gkl/native/libgkl_compression.so
[Wed May 07 21:13:17 MSK 2025] CreateSequenceDictionary --OUTPUT /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.dict --REFERENCE /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna --TRUNCATE_NAMES_AT_WHITESPACE true --NUM_SEQUENCES 2147483647 --VERBOSITY INFO --QUIET false --VALIDATION_STRINGENCY STRICT --COMPRESSION_LEVEL 2 --MAX_RECORDS_IN_RAM 500000 --CREATE_INDEX false --CREATE_MD5_FILE false --GA4GH_CLIENT_SECRETS client_secrets.json --help false --version false --showHidden false --USE_JDK_DEFLATER false --USE_JDK_INFLATER false
[Wed May 07 21:13:17 MSK 2025] Executing as ivan@ivan-Redmi-Book-Pro-15-2022 on Linux 6.11.0-25-generic amd64; OpenJDK 64-Bit Server VM 11.0.13+7-b1751.21; Deflater: Intel; Inflater: Intel; Provider GCS is available; Picard version: Version:4.3.0.0
[Wed May 07 21:13:44 MSK 2025] picard.sam.CreateSequenceDictionary done. Elapsed time: 0.45 minutes.
Runtime.totalMemory()=490733568
Tool returned:
0

по итогу у нас из референсов должны получится 
✅ Final checklist before re-running GATK:

    ✅ GCF_000001405.40_GRCh38.p14_genomic.fna

    ✅ GCF_000001405.40_GRCh38.p14_genomic.fna.fai

    ✅ GCF_000001405.40_GRCh38.p14_genomic.dict

И это еще не вся песня 
ВОТ ТАКАЯ ОШИБКА ВЫЛЕЗЛА 
A USER ERROR has occurred: Input files reference and features have incompatible contigs: No overlapping contigs found.
  reference contigs = [NC_000001.11, NT_187361.1, NT_187362.1, NT_187363.1, NT_187364.1, NT_187365.1, NT_187366.1, NT_187367.1, NT_187368.1, NT_187369.1, NC_000002.12, NT_187370.1, NT_187371.1, NC_000003.12, NT_167215.1, NC_000004.12, NT_113793.3, NC_000005.10, NT_113948.1, NC_000006.12, NC_000007.14, NC_000008.11, NC_000009.12, NT_187372.1, NT_187373.1, NT_187374.1, NT_187375.1, NC_000010.11, NC_000011.10, NC_000012.12, NC_000013.11, NC_000014.9, NT_113796.3, NT_167219.1, NT_187377.1, NT_113888.1, NT_187378.1, NT_187379.1, NT_187380.1, NT_187381.1, NC_000015.10, NT_187382.1, NC_000016.10, NT_187383.1, NC_000017.11, NT_113930.2, NT_187384.1, NT_187385.1, NC_000018.10, NC_000019.10, NC_000020.11, NC_000021.9, NC_000022.11, NT_187386.1, NT_187387.1, NT_187388.1, NT_187390.1, NT_187391.1, NT_187392.1, NT_187393.1, NT_187394.1, NC_000023.11, NC_000024.10, NT_187395.1, NT_187396.1, NT_187397.1, NT_187398.1, NT_187399.1, NT_187400.1, NT_187401.1, NT_187402.1, NT_187403.1, NT_187404.1, NT_187405.1, NT_187406.1, NT_187407.1, NT_187408.1, NT_187409.1, NT_187410.1, NT_187411.1, NT_187412.1, NT_187413.1, NT_187414.1, NT_187415.1, NT_187416.1, NT_187417.1, NT_187418.1, NT_187419.1, NT_187420.1, NT_187421.1, NT_187422.1, NT_187423.1, NT_187424.1, NT_187425.1, NT_187426.1, NT_187427.1, NT_187428.1, NT_187429.1, NT_187430.1, NT_187431.1, NT_187432.1, NT_187433.1, NT_187434.1, NT_187435.1, NT_187436.1, NT_187437.1, NT_187438.1, NT_187439.1, NT_187440.1, NT_187441.1, NT_187442.1, NT_187443.1, NT_187444.1, NT_187445.1, NT_187446.1, NT_187447.1, NT_187448.1, NT_187449.1, NT_187450.1, NT_187451.1, NT_187452.1, NT_187453.1, NT_187454.1, NT_187455.1, NT_187456.1, NT_187457.1, NT_187458.1, NT_187459.1, NT_187460.1, NT_187461.1, NT_187462.1, NT_187463.1, NT_187464.1, NT_187465.1, NT_187466.1, NT_187467.1, NT_187468.1, NT_187469.1, NT_187470.1, NT_187471.1, NT_187472.1, NT_187473.1, NT_187474.1, NT_187475.1, NT_187476.1, NT_187477.1, NT_187478.1, NT_187479.1, NT_187480.1, NT_187481.1, NT_187482.1, NT_187483.1, NT_187484.1, NT_187485.1, NT_187486.1, NT_187487.1, NT_187488.1, NT_187489.1, NT_187490.1, NT_187491.1, NT_187492.1, NT_187493.1, NT_187494.1, NT_187495.1, NT_187496.1, NT_113901.1, NT_167213.1, NT_167214.1, NT_167218.1, NT_187497.1, NT_167220.1, NT_167208.1, NT_187498.1, NT_187499.1, NT_187500.1, NT_187501.1, NT_187502.1, NT_187503.1, NT_187504.1, NT_187505.1, NT_187506.1, NT_187508.1, NT_187509.1, NT_187510.1, NT_187511.1, NT_187512.1, NT_167209.1, NT_187513.1, NT_167211.2, NT_113889.1, NW_012132914.1, NW_015495298.1, NW_025791756.1, NW_011332688.1, NW_014040926.1, NW_009646195.1, NW_018654706.1, NW_019805487.1, NW_009646194.1, NW_018654707.1, NW_014040925.1, NW_017852928.1, NW_009646196.1, NW_025791753.1, NW_025791758.1, NW_025791759.1, NW_025791754.1, NW_011332687.1, NW_018654708.1, NW_014040927.1, NW_025791757.1, NW_025791755.1, NW_021159988.1, NW_025791767.1, NW_025791768.1, NW_025791766.1, NW_025791763.1, NW_012132915.1, NW_025791760.1, NW_025791765.1, NW_018654709.1, NW_025791762.1, NW_025791761.1, NW_025791764.1, NW_015495299.1, NW_018654710.1, NW_011332690.1, NW_021159987.1, NW_011332689.1, NW_017363813.1, NW_025791770.1, NW_025791771.1, NW_009646197.1, NW_012132916.1, NW_011332691.1, NW_021159989.1, NW_018654711.1, NW_012132917.1, NW_009646198.1, NW_019805491.1, NW_019805492.1, NW_019805490.1, NW_019805489.1, NW_019805488.1, NW_025791769.1, NW_021159990.1, NW_021159993.1, NW_013171799.1, NW_025791774.1, NW_021159992.1, NW_021159994.1, NW_021159991.1, NW_021159995.1, NW_013171800.1, NW_013171801.1, NW_025791772.1, NW_017363814.1, NW_025791773.1, NW_015495301.1, NW_015495300.1, NW_025791777.1, NW_018654712.1, NW_025791776.1, NW_009646199.1, NW_016107297.1, NW_025791779.1, NW_021159996.1, NW_025791778.1, NW_016107298.1, NW_025791775.1, NW_018654713.1, NW_013171803.1, NW_025791780.1, NW_012132918.1, NW_009646200.1, NW_013171802.1, NW_017363815.1, NW_021159997.1, NW_021159998.1, NW_019805493.1, NW_025791781.1, NW_017852929.1, NW_017852930.1, NW_018654714.1, NW_018654715.1, NW_012132919.1, NW_025791785.1, NW_018654717.1, NW_017852932.1, NW_025791782.1, NW_017852931.1, NW_025791784.1, NW_019805494.1, NW_025791786.1, NW_018654716.1, NW_025791783.1, NW_013171804.1, NW_013171805.1, NW_025791789.1, NW_025791787.1, NW_025791788.1, NW_009646201.1, NW_021159999.1, NW_021160000.1, NW_011332694.1, NW_021160001.1, NW_013171806.1, NW_009646202.1, NW_013171807.1, NW_025791790.1, NW_011332693.1, NW_011332692.1, NW_015148966.2, NW_025791792.1, NW_021160004.1, NW_025791794.1, NW_011332695.1, NW_021160006.1, NW_019805496.1, NW_019805495.1, NW_017363816.1, NW_025791793.1, NW_019805498.1, NW_021160005.1, NW_021160003.1, NW_019805497.1, NW_013171808.1, NW_021160002.1, NW_025791791.1, NW_009646203.1, NW_013171809.1, NW_018654718.1, NW_021160008.1, NW_011332696.1, NW_009646204.1, NW_025791795.1, NW_018654720.1, NW_015148967.1, NW_018654719.1, NW_011332697.1, NW_019805499.1, NW_021160007.1, NW_021160012.1, NW_011332699.1, NW_013171810.1, NW_021160009.1, NW_009646205.1, NW_011332700.1, NW_013171811.1, NW_021160010.1, NW_021160011.1, NW_011332698.1, NW_021160013.1, NW_025791796.1, NW_018654722.1, NW_021160014.1, NW_018654721.1, NW_011332701.1, NW_021160018.1, NW_021160017.1, NW_012132920.1, NW_025791798.1, NW_021160016.1, NW_025791797.1, NW_021160015.1, NW_025791799.1, NW_013171812.1, NW_019805500.1, NW_017852933.1, NW_021160019.1, NW_013171813.1, NW_018654723.1, NW_012132921.1, NW_025791800.1, NW_017363817.1, NW_021160020.1, NW_016107299.1, NW_017363819.1, NW_025791803.1, NW_025791801.1, NW_017363818.1, NW_019805501.1, NW_025791806.1, NW_025791802.1, NW_025791805.1, NW_021160021.1, NW_025791804.1, NW_019805503.1, NW_014040928.1, NW_019805502.1, NW_013171814.1, NW_018654724.1, NW_025791809.1, NW_025791810.1, NW_025791807.1, NW_021160022.1, NW_014040929.1, NW_025791808.1, NW_009646206.1, NW_016107300.1, NW_016107301.1, NW_016107302.1, NW_016107303.1, NW_016107304.1, NW_016107305.1, NW_016107306.1, NW_016107307.1, NW_016107308.1, NW_016107309.1, NW_016107310.1, NW_016107311.1, NW_016107313.1, NW_016107314.1, NW_016107312.1, NW_025791811.1, NW_025791812.1, NW_021160023.1, NW_025791813.1, NW_025791814.1, NW_025791815.1, NW_021160026.1, NW_021160024.1, NW_009646207.1, NW_014040930.1, NW_014040931.1, NW_009646208.1, NW_015148968.1, NW_021160025.1, NW_015148969.2, NW_017363820.1, NW_021160031.1, NW_025791820.1, NW_021160028.1, NW_025791816.1, NW_021160029.1, NW_025791817.1, NW_021160027.1, NW_025791819.1, NW_021160030.1, NW_025791818.1, NW_018654725.1, NW_025791821.1, NW_018654726.1, NW_009646209.1, NT_187515.1, NT_187517.1, NT_187514.1, NT_187520.1, NW_003315905.1, NW_003315906.1, NW_003315907.2, NT_187521.1, NT_187519.1, NT_187516.1, NT_187518.1, NT_187525.1, NT_187526.1, NT_187529.1, NT_187522.1, NW_003315908.1, NT_187524.1, NT_187531.1, NT_187530.1, NT_187528.1, NW_003571033.2, NW_003315909.1, NT_187527.1, NT_187523.1, NW_003871060.2, NT_187535.1, NT_187537.1, NW_003315913.1, NT_187533.1, NT_187536.1, NT_187538.1, NT_187532.1, NT_187534.1, NT_187539.1, NT_187540.1, NW_003315915.1, NT_187541.1, NT_167250.2, NT_187544.1, NW_003315914.1, NT_187542.1, NT_187545.1, NT_187543.1, NT_187550.1, NT_187548.1, NT_187547.1, NW_003315920.1, NW_003571036.1, NT_187551.1, NW_003315917.2, NW_003315918.1, NT_187549.1, NW_003315919.1, NT_187546.1, NT_167244.2, NT_187555.1, NT_187554.1, NW_003315921.1, NT_187556.1, NT_187557.1, NW_004166862.2, NT_187552.1, NT_187553.1, NT_187558.1, NT_187561.1, NT_187559.1, NW_003315922.2, NT_187562.1, NT_187564.1, NT_187563.1, NT_187560.1, NT_187572.1, NT_187568.1, NT_187565.1, NT_187576.1, NT_187570.1, NT_187577.1, NT_187566.1, NT_187567.1, NT_187574.1, NT_187575.1, NT_187573.1, NT_187571.1, NT_187569.1, NW_003315928.1, NW_003315929.1, NW_003315930.1, NW_003315931.1, NT_187578.1, NW_003315934.1, NT_187579.1, NW_003315935.1, NT_187586.1, NT_187584.1, NT_187585.1, NT_187583.1, NW_003315936.1, NW_003871073.1, NW_003871074.1, NT_187582.1, NT_187581.1, NW_003571049.1, NW_003571050.1, NT_187588.1, NW_003315938.1, NT_187587.1, NW_003315939.2, NW_003315941.1, NW_003315942.2, NT_187590.1, NW_003315940.1, NT_187589.1, NT_187591.1, NT_187594.1, NT_187593.1, NT_187597.1, NT_187595.1, NT_187592.1, NT_187596.1, NT_187598.1, NT_187601.1, NT_187599.1, NT_187600.1, NT_187602.1, NT_187604.1, NT_187603.1, NW_003315943.1, NT_187605.1, NW_003315944.2, NT_187606.1, NT_187610.1, NT_187609.1, NT_187608.1, NT_187607.1, NW_003315945.1, NW_003315946.1, NW_003315952.3, NT_187613.1, NT_187611.1, NT_187614.1, NW_003871091.1, NW_003871092.1, NW_003315953.2, NT_167251.2, NW_003315954.1, NT_187615.1, NT_187616.1, NW_003315955.1, NT_187612.1, NT_187618.1, NW_003315956.1, NW_003315959.1, NW_003315960.1, NW_003315957.1, NW_003315958.1, NW_003315961.1, NT_187617.1, NT_187622.1, NT_187621.1, NW_003315962.1, NW_003315964.2, NW_003315965.1, NW_003315963.1, NT_187619.1, NT_187620.1, NW_003571054.1, NW_003315966.2, NT_187623.1, NT_187625.1, NT_187624.1, NW_003315967.2, NT_187628.1, NT_187627.1, NW_003315968.2, NW_003315969.2, NW_003315970.2, NT_187626.1, NT_187629.1, NT_187632.1, NT_187633.1, NT_187630.1, NT_187631.1, NW_003315972.2, NW_003315971.2, NT_187634.1, NT_187635.1, NT_187646.1, NT_187648.1, NT_187647.1, NT_187649.1, NT_187650.1, NT_187651.1, NT_187652.1, NT_113891.3, NT_187653.1, NT_187655.1, NT_187654.1, NT_187656.1, NT_187657.1, NT_187658.1, NT_187659.1, NT_187660.1, NT_187662.1, NT_187664.1, NT_187661.1, NW_003871093.1, NT_187663.1, NT_187665.1, NT_187666.1, NW_003571055.2, NW_004504305.1, NT_187667.1, NT_187678.1, NT_187679.1, NT_167245.2, NT_187680.1, NT_187681.1, NW_003571056.2, NT_187682.1, NT_187688.1, NT_167246.2, NW_003571057.2, NT_187689.1, NT_167247.2, NW_003571058.2, NT_187690.1, NT_167248.2, NW_003571059.2, NT_187691.1, NT_167249.2, NW_003571060.1, NT_187692.1, NW_003571061.2, NT_187693.1, NT_187636.1, NT_187637.1, NT_187638.1, NT_187639.1, NT_187640.1, NT_187641.1, NT_187642.1, NT_187643.1, NT_187644.1, NT_187645.1, NT_187668.1, NT_187669.1, NT_187670.1, NT_187671.1, NT_187672.1, NT_187673.1, NT_187674.1, NT_187675.1, NT_187676.1, NT_187677.1, NT_187683.1, NT_187684.1, NT_187685.1, NT_187686.1, NT_187687.1, NT_113949.2, NC_012920.1]
  features contigs = [chr1, chr2, chr3, chr4, chr5, chr6, chr7, chr8, chr9, chr10, chr11, chr12, chr13, chr14, chr15, chr16, chr17, chr18, chr19, chr20, chr21, chr22, chrX, chrY]

***********************************************************************
Set the system property GATK_STACKTRACE_ON_USER_EXCEPTION (--java-options '-DGATK_STACKTRACE_ON_USER_EXCEPTION=true') to print the stack trace.

как ее исправлять 

Говорят по хорошему нудно использовать референс для которого у нас есть точно опредленные SNP зашел я в этот файл и там в headerе есть такая строчка 

##fileformat=VCFv4.1
##SomaticSeq=v2.8.1-SEQC2
##truth_set=v1.2
##reference=file:/SEQC2_Resources/GRCh38.d1.vd1.fa
##FILTER=<ID=PASS,Description="Calls considered to be a part of the truth set">
##FILTER=<ID=HighConf,Description="highly confident that it is a real somatic mutation">
##FILTER=<ID=MedConf,Description="confident that it is a real somatic mutation">

ну и спросил у чата гпт где я могу скачать его он мне сказал тут(https://gdc.cancer.gov/about-data/gdc-data-processing/gdc-reference-files)

![image](https://github.com/user-attachments/assets/595d8e25-b829-4e03-bf29-06d8295a1387)


# Для BAM:
samtools view -H input.bam | grep "@SQ" | awk '{print $2}' | sed 's/SN://' | awk -F '.' '{print $0 "\tchr" $2}' > chr_name_mapping.txt

💡 Что делает эта команда:

    samtools view -H input.bam — получаем заголовок.

    grep "@SQ" — выбираем строки с описанием хромосом.

    awk '{print $2}' — берем только поле SN:<name>.

    sed 's/SN://' — удаляем префикс SN:.

    awk -F '.' '{print $0 "\tchr" $2}' — разделяем по точке и берём вторую часть (число после точки), добавляем chr.

получаем файлик такого формата: 

NT_187529.1	chr1
NT_187522.1	chr1
NW_003315908.1	chr1
NT_187524.1	chr1
NT_187531.1	chr1
NT_187530.1	chr1
NT_187528.1	chr1
NW_003571033.2	chr2
NW_003315909.1	chr1

далее 

bcftools annotate --rename-chrs chr_name_mapping.txt input.vcf -o output.vcf


a. Генерация таблицы рекалибровки:
bash

gatk BaseRecalibrator \
   -I deduplicated.bam \
   -R /path/to/hg19.fasta \
   --known-sites /path/to/dbsnp.vcf.gz \
   --known-sites /path/to/1000G_phase1.snps.high_confidence.hg19.vcf \
   -O recal_data.table

реальная команда 

gatk BaseRecalibrator \
   -I SRR7890879_aligned_deduplicated.bam \
   -R /media/ivan/KINGSTON/self_project/reference/GCF_000001405.40_GRCh38.p14_genomic.fna \
   --known-sites /media/ivan/KINGSTON/self_project/raw_data/SEQC2/high-confidence_sINDEL_in_HC_regions_v1.2.1.vcf.gz \
   --known-sites /media/ivan/KINGSTON/self_project/raw_data/SEQC2/high-confidence_sSNV_in_HC_regions_v1.2.1.vcf.gz \
   -O recal_data.table

а еще для каждого референсного файла с SNP у нас должны быть индексные файлы 

gatk IndexFeatureFile \
   -I /media/ivan/KINGSTON/self_project/raw_data/SEQC2/high-confidence_sINDEL_in_HC_regions_v1.2.1.vcf.gz

gatk IndexFeatureFile \
   -I /media/ivan/KINGSTON/self_project/raw_data/SEQC2/high-confidence_sSNV_in_HC_regions_v1.2.1.vcf.gz


b. Применение рекалибровки:
bash

gatk ApplyBQSR \
   -R /path/to/hg19.fasta \
   -I deduplicated.bam \
   --bqsr-recal-file recal_data.table \
   -O recalibrated.bam

Где:

    --known-sites — файлы с известными вариантами (например, dbSNP).

    recalibrated.bam — итоговый BAM-файл после рекалибровки.





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

кто знал кто знал что понадобится еще и --germline-resource {input.gnomad} вот этот файл откуда-то брать 
чат гпт сказал что вот отсюда можно взять и они совместимы будут с любым рефернесмо у кого названия хромсом "chrX"

➡️ GATK рекомендует использовать af-only-gnomad.hg38.vcf.gz
Скачать можно с Broad Institute:

wget https://storage.googleapis.com/gatk-best-practices/somatic-hg38/af-only-gnomad.hg38.vcf.gz
wget https://storage.googleapis.com/gatk-best-practices/somatic-hg38/af-only-gnomad.hg38.vcf.gz.tbi


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





___
рубрика вопросы 
![image](https://github.com/user-attachments/assets/d7ab7f04-4c0f-4b0c-b591-3c59519bcfd7)
почему эти ребята не указывают версии, например sambamba или bwa mem

___
рубрика не знаю куда это засунуть 

вот это я еще раз индексировал 
команда: 
bwa index GRCh38.d1.vd1.fa.tar.gz

вывод команды: 
...

еще раз выравнивал
команда:
(base) ivan@ivan-Redmi-Book-Pro-15-2022:/media/ivan/KINGSTON/self_project/bams$ bwa mem -t 12 /media/ivan/KINGSTON/self_project/reference_SEQC2/GRCh38.d1.vd1.fa.tar.gz /media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_1.fastq.gz /media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_2.fastq.gz | samtools view -b - > SRR7890879_aligned_GRCh38_d1_vd1.bam && { echo -e "\a"; notify-send "BWA Index" "Ура нафиг!"; }

вывод команды 
...

И про это я не забыл 
(sambamba_env) ivan@ivan-Redmi-Book-Pro-15-2022:/media/ivan/KINGSTON/self_project/bams$ sambamba markdup -t 12 SRR7890879_aligned_GRCh38_d1_vd1.bam SRR7890879_aligned_dedup_GRCh38_d1_vd1.bam 
...

bwa index GRCh38.d1.vd1.fa
bwa mem -t 12 /media/ivan/KINGSTON/self_project/reference_SEQC2/GRCh38.d1.vd1.fa /media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_1.fastq.gz /media/ivan/KINGSTON/self_project/raw_data/SEQC2/SRR7890879.sralite.1_2.fastq.gz | samtools view -b - > SRR7890879_aligned_GRCh38_d1_vd1.bam && { echo -e "\a"; notify-send "BWA Index" "Ура нафиг!"; }
conda activate sambamba_env
sambamba markdup -t 12 SRR7890879_aligned_GRCh38_d1_vd1.bam SRR7890879_aligned_dedup_GRCh38_d1_vd1.bam 

     
