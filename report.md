
---

original here:
https://github.com/MPHRS/Genomic-data-analysis/blob/main/Project_6_baking/Report.md

# Differential expression of genes associated with carbohydrate metabolism and transport in Saccharomyces cerevisiae under fermentation conditions
---

# **Abstract**


in this work we‐analyzed RNA‐seq data from _Saccharomyces cerevisiae_ during early dough fermentation (0 vs. 30 min) to uncover transcriptional shifts in carbohydrate metabolism and transport. Alignment to the S288C genome and DESeq2 analysis identified 3,450 differentially expressed genes (padj < 0.05), of which 50 were GO‐Slim annotated. Key glycolytic genes (PGI1, HXK2, PFK27, RGT2) were strongly up-regulated, while the gluconeogenic gene PCK1 was strongly repressed. These coordinated changes highlight yeast’s rapid transition from glucose synthesis to fermentative metabolism, optimizing ATP generation.

---

# **Introduction**

 
_Saccharomyces cerevisiae_ is the key element  in bread making, converting sugars to carbon dioxide and ethanol through anaerobic fermentation. This process begins immediately upon dough mixing, when yeast cells encounter high concentrations of glucose and limited oxygen. To thrive under these conditions, _S. cerevisiae_ must rapidly reprogram its metabolism.

To establish these early transcriptional changes, we re‐analyzed publicly available RNA-seq data from cells sampled at 0 and 30 minutes of dough fermentation. Reads were normalized for library size and modeled as negative-binomial counts, allowing robust estimation of gene‐specific variance and well‐calibrated testing for differential expression—even with only two replicates per time point. Genes with adjusted p-values below 0.05 were considered significant and subjected to GO-Slim mapping to reveal the key regulatory and metabolic nodes driving the initial adaptive response.

Accurate detection of differential expression from RNA-Seq requires accounting for both sampling noise and true biological variability. We applied this approach via DESeq2 to compare yeast transcript levels at 0 min versus 30 min of dough fermentation. [1]

# **Methods**
The analysis utilized publicly available RNA-seq data from Saccharomyces cerevisiae fermentation experiments (SRA accessions SRR941816-SRR941819), which represent two biological replicates at 0-minute and 30-minute fermentation time points. The reference genome for strain S288C (assembly GCF_000146045.2_R64) and its annotation GFF file were obtained from NCBI, with additional genomic context provided by the Saccharomyces Genome Database (SGD).

Read alignment was performed using HISAT2 (v2.2.1) with default parameters after genome indexing. Resulting BAM files were sorted via SAMtools (v1.15). Gene expression quantification was conducted using featureCounts from the Subread package (v2.0.8) with the -g gene_id parameter. Prior to analysis, GFF annotations were converted to GTF format using gffread (v0.12.7), followed by attribute corrections based on guidelines from a public Colab notebook.

Differential expression analysis was implemented in R (v4.3.1) using the DESeq2 package (v1.38.3), including normalization and statistical processing stages. Functional annotation of the top 50 differentially expressed genes was performed via the GO Slim Mapper online tool with default settings . Results were visualized using custom python scripts provided in the supplementary materials.

The complete analysis scripts and intermediate files are available in the laboratory notebook.


# **Results**

The total number of reads per sample ranged from 1.7 to 9.9 million, with the proportion of successfully aligned reads ranging from 94.3% to 96.2%. In all samples, more than 87% of reads were mapped to a single genomic location, while the proportion of reads not aligned to any position did not exceed 6%.

During the work, top 50 genes sorted by adjusted p-values(i.e. significant differential expression) were selected. Functional annotation allowed for their distribution into Gene Ontology (GO) categories. For each category, both the number of genes present among the selected 50 and their total representation in the annotated genome were calculated. Summary data are presented in Table 1.

Table 1. Distribution of differentially expressed genes by GO categories.


| **GO Term (GO ID)**                                                     | **Genes Annotated to the GO Term**                                                                                      | **GO Term Usage in Gene List** | **Genome Frequency of Use** |     |
| :---------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------- | -----------------------------: | --------------------------: | --- |
| **rRNA processing** (GO:0006364)                                        | `YDR449C`, `YEL026W`, `YGR159C`, `YHR066W`, `YHR196W`, `YJL069C`, `YMR093W`, `YNL112W`, `YNL182C`, `YOL041C`, `YOL080C` |             **11/48** (22.92%) |        **329/6489** (5.07%) |     |
| **ribosomal large subunit biogenesis** (GO:0042273)                     | `YCR072C`, `YDL063C`, `YHR066W`, `YIR012W`, `YJL122W`, `YNL182C`, `YOL041C`, `YOL080C`                                  |              **8/48** (16.67%) |        **126/6489** (1.94%) |     |
| **transcription by RNA polymerase I** (GO:0006360)                      | `YHR196W`, `YJL148W`, `YJR063W`, `YML043C`, `YMR093W`, `YNL248C`                                                        |              **6/48** (12.50%) |         **69/6489** (1.06%) |     |
| **ribosomal small subunit biogenesis** (GO:0042274)                     | `YDR449C`, `YEL026W`, `YGR159C`, `YHR196W`, `YJL069C`, `YMR093W`                                                        |              **6/48** (12.50%) |        **135/6489** (2.08%) |     |
| **ribosome assembly** (GO:0042255)                                      | `YCR072C`, `YGR159C`, `YHR066W`, `YIR012W`, `YNL182C`, `YOL080C`                                                        |              **6/48** (12.50%) |         **77/6489** (1.19%) |     |
| **transmembrane transport** (GO:0055085)                                | `YDR536W`, `YHR094C`, `YKL120W`, `YNL065W`                                                                              |               **4/48** (8.33%) |        **278/6489** (4.28%) |     |
| **nucleobase-containing small molecule metabolic process** (GO:0055086) | `YBL039C`, `YMR300C`, `YNL141W`, `YOL136C`                                                                              |               **4/48** (8.33%) |        **178/6489** (2.74%) |     |
| **carbohydrate metabolic process** (GO:0005975)                         | `YBR105C`, `YER062C`, `YKR097W`, `YOL136C`                                                                              |               **4/48** (8.33%) |        **160/6489** (2.47%) |     |
| **RNA catabolic process** (GO:0006401)                                  | `YLR264W`, `YNL112W`, `YOR359W`                                                                                         |               **3/48** (6.25%) |        **162/6489** (2.50%) |     |
| **DNA-templated transcription elongation** (GO:0006354)                 | `YJL148W`, `YNL248C`                                                                                                    |               **2/48** (4.17%) |        **105/6489** (1.62%) |     |
| **tRNA processing** (GO:0008033)                                        | `YOL124C`, `YPL212C`                                                                                                    |               **2/48** (4.17%) |        **119/6489** (1.83%) |     |
| **transcription by RNA polymerase II** (GO:0006366)                     | `YJR063W`, `YNL112W`                                                                                                    |               **2/48** (4.17%) |        **489/6489** (7.54%) |     |
| **regulation of DNA metabolic process** (GO:0051052)                    | `YNL182C`, `YOR359W`                                                                                                    |               **2/48** (4.17%) |        **115/6489** (1.77%) |     |
| **proteolysis involved in protein catabolic process** (GO:0051603)      | `YBR105C`, `YLR224W`                                                                                                    |               **2/48** (4.17%) |        **225/6489** (3.47%) |     |
| **RNA modification** (GO:0009451)                                       | `YOL124C`, `YPL212C`                                                                                                    |               **2/48** (4.17%) |        **175/6489** (2.70%) |     |
| **DNA-templated transcription termination** (GO:0006353)                | `YJR063W`, `YNL112W`                                                                                                    |               **2/48** (4.17%) |         **42/6489** (0.65%) |     |
| **carbohydrate transport** (GO:0008643)                                 | `YDR536W`, `YHR094C`                                                                                                    |               **2/48** (4.17%) |         **36/6489** (0.55%) |     |
| **lipid metabolic process** (GO:0006629)                                | `YBL039C`, `YOL151W`                                                                                                    |               **2/48** (4.17%) |        **297/6489** (4.58%) |     |
| **ribosomal subunit export from nucleus** (GO:0000054)                  | `YLR264W`                                                                                                               |               **1/48** (2.08%) |         **67/6489** (1.03%) |     |
| **regulation of organelle organization** (GO:0033043)                   | `YLR180W`                                                                                                               |               **1/48** (2.08%) |        **227/6489** (3.50%) |     |
| **amino acid transport** (GO:0006865)                                   | `YNL065W`                                                                                                               |               **1/48** (2.08%) |         **45/6489** (0.69%) |     |
| **monoatomic ion transport** (GO:0006811)                               | `YNR060W`                                                                                                               |               **1/48** (2.08%) |         **99/6489** (1.53%) |     |
| **tRNA aminoacylation for protein translation** (GO:0006418)            | `YDR037W`                                                                                                               |               **1/48** (2.08%) |         **36/6489** (0.55%) |     |
| **amino acid metabolic process** (GO:0006520)                           | `YLR180W`                                                                                                               |               **1/48** (2.08%) |        **157/6489** (2.42%) |     |
| **DNA recombination** (GO:0006310)                                      | `YGR159C`                                                                                                               |               **1/48** (2.08%) |        **194/6489** (2.99%) |     |
| **response to chemical** (GO:0042221)                                   | `YOR271C`                                                                                                               |               **1/48** (2.08%) |        **414/6489** (6.38%) |     |
| **RNA splicing** (GO:0008380)                                           | `YEL026W`                                                                                                               |               **1/48** (2.08%) |        **139/6489** (2.14%) |     |
| **organelle assembly** (GO:0070925)                                     | `YLR180W`                                                                                                               |               **1/48** (2.08%) |        **116/6489** (1.79%) |     |
| **protein targeting** (GO:0006605)                                      | `YBR105C`                                                                                                               |               **1/48** (2.08%) |        **171/6489** (2.64%) |     |
| **DNA replication** (GO:0006260)                                        | `YNL182C`                                                                                                               |               **1/48** (2.08%) |        **139/6489** (2.14%) |     |
| **generation of precursor metabolites and energy** (GO:0006091)         | `YOL136C`                                                                                                               |               **1/48** (2.08%) |         **84/6489** (1.29%) |     |
| **response to osmotic stress** (GO:0006970)                             | `YER062C`                                                                                                               |               **1/48** (2.08%) |         **88/6489** (1.36%) |     |
| **monocarboxylic acid metabolic process** (GO:0032787)                  | `YOL136C`                                                                                                               |               **1/48** (2.08%) |        **132/6489** (2.03%) |     |
| **nucleobase-containing compound transport** (GO:0015931)               | `YHR196W`                                                                                                               |               **1/48** (2.08%) |        **121/6489** (1.86%) |     |
| **cytoplasmic translation** (GO:0002181)                                | `YLR264W`                                                                                                               |               **1/48** (2.08%) |        **169/6489** (2.60%) |     |
| **mRNA processing** (GO:0006397)                                        | `YEL026W`                                                                                                               |               **1/48** (2.08%) |        **170/6489** (2.62%) |     |

The analysis showed that a significant portion of the differentially expressed genes are related to fundamental cellular metabolic processes, including ribosome biogenesis, transcription, and substance transport.
Genes involved in carbohydrate metabolism are of particular interest in the context of our study. The analysis highlighted several carbohydrate‐related GO‐Slim terms, each represented by a small set of genes:

- **Carbohydrate metabolic process (GO:0005975):** YBR105C, YER062C, YKR097W, YOL136C
    
- **Carbohydrate transport (GO:0008643):** YDR536W, YHR094C
    
- **Generation of precursor metabolites and energy (GO:0006091):** YOL136C
    
- **Monocarboxylic acid metabolic process (GO:0032787):** YOL136C

Five out of the six analyzed genes, some of which are simultaneously involved in multiple processes, demonstrated a statistically significant increase in expression (log2FoldChange > 2.5, padj < 0.001), while the YKR097W gene showed the opposite trend with a marked decrease in expression levels (Fig. 1).

![image](https://github.com/user-attachments/assets/471c6256-1c3d-4794-9f4b-0c1efa9a3863)




  
# **Discussion**

Our analysis of Saccharomyces cerevisiae RNA-seq data before (0 min) and during (30 min) fermentation in bread dough identified **3450** genes with statistically significant changes in expression(padj < 0.05). Of these, **50 genes** were selected to functional annotation. In the amount of 48 could be associated with **36 distinct GO-Slim biological processes** (Table 1). This represent a wide changing of yeast cellular functions as they shift from aerobic respiration to anaerobic fermentation and glucose availability.

Building on the global shifts we observed, let’s zoom in on **carbohydrate metabolic process (GO:0005975)** and then contrast one clearly **up-regulated** GO term with one **down-regulated** GO term, to see how they contribute to the yeast’s metabolic switch.

table 2. genes differential expression for carbohydrate metabolic process.

| Gene                   | Function                                                                                 | log₂FC  | Regulation |
| ---------------------- | ---------------------------------------------------------------------------------------- | ------- | ---------- |
| **YBR105C**  <br>VID24 | GID Complex regulatory subunit                                                           | +4.92   | up         |
| **YER062C**  <br>GPP2  | Involved in glycerol biosynthesis                                                        | +7.90   | up         |
| **YKR097W**  <br>PCK1  | Key enzyme in gluconeogenesis                                                            | –4.71   | down       |
| **YOL136C**  <br>PFK27 |  6-Phosphofructo-2-kinase; synthesizes F₂,6-BP to activate glycolysis                    | +5.71   | up         |
| **YDR536W** STL1       | Glycerol proton symporter (osmosensing response; transiently induced upon osmotic shock) | +7.8760 | up         |
| **YHR094C** HXT1           | Low-affinity glucose transporter (high-capacity uptake at elevated glucose levels)       | +7.8836 | up         |

**Down-regulation of PCK1**  
PCK1 encodes phosphoenolpyruvate carboxykinase, the rate-limiting enzyme of gluconeogenesis. When extracellular glucose becomes plentiful, the cell abandons glucose synthesis in favor of high-flux glycolysis. Glucose induces a hierarchical repression cascade that shuts down transcription of gluconeogenic genes, and simultaneously accelerates degradation of existing PCK1 mRNAs. Together, these mechanisms rapidly remove Pck1p and its transcript, preventing futile glucose production under fermentative conditions[2].

**Up-regulation of VID24, PFK27 and GPP2**  
While gluconeogenic capacity is dismantled, yeast boosts three complementary activities to optimize fermentative growth:

- **VID24**: Elevated VID24 expression enhances the GID ubiquitin-ligase’s ability to target fructose-1,6-bisphosphatase (FBPase) for proteasomal degradation.     
- **PFK27**: Up-regulation of PFK27 raises intracellular fructose-2,6-bisphosphate levels, a potent allosteric activator of phosphofructokinase-1. This amplifies the glycolytic flux needed for rapid ATP generation under anaerobic, glucose-replete conditions.    
- **GPP2**: Increased GPP2 expression diverts excess triose phosphates into glycerol synthesis. Glycerol production not only regenerates NAD+(essential part for sustained glycolysis), but also helps maintain osmotic balance in the high-sugar environment of dough.    


Taken together, yeast runs a precise program: it breaks down gluconeogenic enzymes and their mRNAs, and turns on genes for glycolysis and stress protection. This balance of protein breakdown and gene activation lets _S. cerevisiae_ switch quickly into anaerobic fermentation. As a result, it maximizes ATP production and keeps its redox and osmotic balance in high-sugar dough.

# **Supplementary materials**

Link to colab notebook: https://colab.research.google.com/drive/1yjUwu6Xey68g-GfQKJA9SoBX6RwVZk_r?usp=sharing

Lab notebook: https://github.com/MPHRS/Genomic-data-analysis/blob/main/Project_6_baking/Lab_notebook.md
# References

1. Anders, S., Huber, W. Differential expression analysis for sequence count data. _Genome Biol_ **11**, R106 (2010). https://doi.org/10.1186/gb-2010-11-10-r106
2. Méndez-Lucas A, Hyroššová P, Novellasdemunt L, Viñals F, Perales JC. Mitochondrial phosphoenolpyruvate carboxykinase (PEPCK-M) is a pro-survival, endoplasmic reticulum (ER) stress response gene involved in tumor cell adaptation to nutrient availability. J Biol Chem. 2014 Aug 8;289(32):22090-102. doi: 10.1074/jbc.M114.566927. Epub 2014 Jun 27. PMID: 24973213; PMCID: PMC4139223.
3. Cherry JM, Hong EL, Amundsen C, Balakrishnan R, Binkley G, Chan ET, et al. Saccharomyces Genome Database: the genomics resource of budding yeast. Nucleic Acids Res. 2012 Jan;40(Database issue):D700–5. doi:10.1093/nar/gkr1029
4. Norbeck J, Pahlman AK, Akhtar N, Blomberg A, Adler L. Purification and characterization of two isoenzymes of DL-glycerol-3-phosphatase from _Saccharomyces cerevisiae_: identification of the corresponding GPP1 and GPP2 genes and evidence for osmotic regulation of Gpp2p expression by the osmosensing mitogen-activated protein kinase signal transduction pathway. J Biol Chem. 1996 Jun 7;271(23):13875–13881. doi:10.1074/jbc.271.23.13875
