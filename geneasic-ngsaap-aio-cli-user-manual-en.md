---
title: GeneASIC NGSAAP All-in-One Command Line Tools User Manual

---

```
Contact information:
GeneASIC Technologies Corp.
info@geneasic.com
```
Version 1. 1 .0_b

# GeneASIC NGSAAP All-in-One Command Line Tools User Manual

## Before reading the User Manual
This user manual is designated for GeneASIC NGSAAP All-in-One v1. 1 .0. Please read through this user manual before using the GeneASIC NGSAAP.

All figures included in this document serve as illustrative examples. Actual software content may vary slightly.
For accurate information, please consult the version of software you have acquired.

GeneASIC Technologies Corp. has implemented steps to ensure the accuracy of this manual. Nonetheless, GeneASIC Technologies Corp. cannot be held liable for any errors or omissions within the manual and retains the authority to update both the manual and the software to enhance their reliability, performance, or design.

Trademarks and brand names mentioned in this manual are the property of their respective owners.

Please contact GeneASIC Technologies Corp. for any further questions.

© 2025 GeneASIC Technologies Corp. All rights reserved.


## Content

- 1. Introduction
   - 1.1 Computing architecture
   - 1.2 Genomic variant analysis workflow
- 2. Example of WGS analysis
   - 2.1 Customize pipeline configuration settings
   - 2.2 Generate a dataset list file
- 3. Details of command line operation
   - 3.1 gconf: Pipeline configuration settings
   - 3.2 gsched: Sample data configuration
   - 3.3 gsub: Job submission
   - 3.4 gwatch: Job queue viewer
   - 3.5 gcancel : Job cancellation
   - 3.6 gversion : Program Version Viewer
   - 3.7 acctmgr : User Account Manager
- 4. Revision History


## 1. Introduction

The GeneASIC NGSAAP (Next-Generation Sequencing Analysis Acceleration Platform) is a hardware-accelerated NGS secondary analysis system based on FPGA (Field-Programmable Gate Array). Beyond its genomic analysis capabilities, the GeneASIC NGSAAP is designed for scalability and ease of use, featuring a streaming architecture that processes data in real-time within a distributed computing framework.

### 1.1 Computing architecture

The GeneASIC NGSAAP features a distributed computing architecture that optimizes the processing of large genomic datasets. Each GeneASIC NGSAAP node is equipped with FPGA acceleration cards to enhance processing speed for tasks such as read mapping and variant calling. This architecture not only accelerates complex genomic analyses but also frees up CPU resources for additional tasks, such as advanced variant discovery, database annotation, and clinical significance interpretation.
![fig1](https://hackmd.io/_uploads/HkOWtuCMZe.png)

Figure 1. GeneASIC NGSAAP computing architecture

Resource allocation and task management are seamlessly coordinated by SLURM, Snakemake, and the GeneASIC pipelining controller, providing scalability for projects of any size. SLURM enables users to enqueue jobs that are automatically dispatched as nodes become available. Together with Snakemake, the pipelining controller optimizes the use of FPGA, CPU, memory, and disk resources, efficiently managing diverse tasks,even on consumer-grade machines. GeneASIC NGSAAP also supports multi-user environments, allowing multiple users to submit jobs concurrently without manual scheduling or node availability concerns. These features significantly enhance GeneASIC NGSAAP’s usability for large-scale projects, maximizing capacity and efficiency in genomic data analysis.

The data flow within GeneASIC NGSAAP allows for direct streaming of input from sources such as USB drives, MicroSD cards, network-attached storage (NAS) or data centers with network file systems (NFS) support. This capability enables concurrent data transfer and processing. Once analysis is complete, results are securely transferred to user-specific storage locations. All operations are conducted on-premises within a local area network (LAN), ensuring maximum genomic data privacy and minimizing the need to prefetch large datasets,such as FASTQ files.

In summary, GeneASIC NGSAAP leverages FPGAs to accelerate core analyses and utilizes SLURM for efficient, comprehensive, secure, and scalable genomic data analysis. Multi-user support and streamlined data flow make GeneASIC NGSAAP a powerful tool for accelerating genomic discoveries without computational bottlenecks.

### 1.2 Genomic variant analysis workflow

The GeneASIC NGSAAP pipeline, powered by FPGA, consists of two main stages for short variant analysis: read mapping and variant calling. In the All-in-One version, additional analyses and variant interpretation are included, such as structural variant analysis, variant annotations, and more.

**Stage 1. Read Mapping**

The process commences with mapping of raw sequencing reads to a reference genome. This stage encompasses crucial preprocessing steps, including read trimming to eliminate low-quality bases or adaptors, the organization of reads by their positional coordinates. The outcome of this stage is a set of analysis-ready reads, setting the stage for subsequent variant calling.

**Stage 2. Variant Calling**

The second stage handles short variant calling with FPGA hardware acceleration. Machine learning techniques are then applied to refine and filter the identified variants. PCR-bias correction is optionally applied, depending on library preparation settings. The output includes a detailed catalog of SNPs and INDELs in VCF format. Additionally, in All-in-One version, this stage also performs CPU-powered analyses such as structural variant calling (Manta), copy number analysis (ASCAT, SLM and SMNCopyNumberCaller), potential uniparental disomy extraction, short tandem repeat size estimation (ExpansionHunter), HLA typing (SpecHLA), clonal hematopoiesis analysis (GATK Mutect2), and mitochondrial variant calling (GATK Mutect2 in mitochondrial mode).

**Stage 3. Variant Interpretation**

In the final stage, variants are annotated and undergo rigorous analysis following ACMP-AMP guidelines for variant classification, whereas structural variants are interpreted with AnnotSV. Moreover, carrier screening and secondary findings recommended by ACMG, pharmacogenomics annotation, and runs of homozygosity (ROH) analysis are also completed in this stage. All results are finally summarized into reports in XLSX and PDF formats.

![fig2](https://hackmd.io/_uploads/S1YQsORfWg.png)

Figure 2. GeneASIC NGSAAP workflow


## 2. Example of WGS analysis

Below is a step-by-step example of how to analyze WGS data.

### Step 1. Prepare workspaces and datasets
```
#creat a working directory
mkdir example
cd example
```

For batch analysis, creating a CSV file that specifies job names with associated paths to FASTQ files is recommended. An example of the CSV, data.csv, for analysis:

```
sample1,fastq/0,/home/egs/sample1_R1.fastq.gz
sample1,fastq/1,/home/egs/sample1_R2.fastq.gz
sample2,fastq/0,/home/egs/sample2_R1.fastq.gz
sample2,fastq/1,/home/egs/sample2_R2.fastq.gz
```
fastq/0 and fastq/1 represent segments 1 and 2 of the paired-end sequencing data, respectively. Make sure the data.csv is stored in a dataset workspace.

### Step 2. Generate a task list
```
#generate a task list with a CSV file
gsched -b batch.yaml\
-p default-germline-wgs-pcrfree -s fastq -t bam vcf\
-i /home/egs -f data.csv -o results -a
```
This command generates a batch.yaml file that associates input data specified in data.csv with the pipeline configuration settings named default-germline-wgs-pcr-free. The -s option indicates that the input requires FASTQ data, while the -t option specifies that the outputs are BAM and VCF file. The -i option sets the import directory for data.csv, and the -o defines the output directory as results. Apply all the settings with the -a option and the command exports the batch.yaml file.



![fig3](https://hackmd.io/_uploads/r1Ag0_Rz-g.png)

Figure 3. Example of batch.yaml content


### Step 3. Submit a task list to the job queue
```
gsub batch.yaml
```
gsub allows to submit task lists to the GeneASIC NGSAAP job queue. A batch ID would be returned if the job submission is successful.

### Step 4. View job status

```
gwatch
```

gwatch allows to view all job status.Upon running this command, a task list in the job queue would be displayed as below. To check the execution details, use the arrow keys and the ENTER to select the specific task you are interested in. By pressing the ESC key, you can exit the job status viewer.

![fig4](https://hackmd.io/_uploads/BJuFRuAzZe.png)

Figure 4. GeneASIC NGSAAP execution details

### Step 5. Inspect results

Once the data analysis is complete, you can review results in the output directory specified in Step 2.
![fig5](https://hackmd.io/_uploads/SJRHJKCMZg.png)
Figure 5. Example of output results

### 2.1 Customize pipeline configuration settings

An appropriate pipeline configuration setting could lead to a better result and varies based on factors such as sequencing platform, data type, library preparation and the balance between speed and accuracy. Below are a few examples for different configurations.

Replace *<identifier>* with an actual identifier. These configurations help customize the analysis settings to meet specific sequencing settings. -a option in gconf saves the configuration settings, allowing them to be easily reused in gsched with the -p option. For more details on the gconf command, please refer to Section 3.1.

- For PCR-based WGS data
```
gconf touch *<identifier>* --sequencing-protocol pcr -a
```
- For WES data
```
gconf touch *<identifier>* --sequencing-datetype wes \
--sequencing-protocol pcr --interval-of-interest \
/opt/geneasic/share/intervals/grch38/*<target-region>*.bed -a
```
- For NovaSeq or other platforms using 2-channel SBS technology
```
gconf touch *<identifier>* --sequencing-platform novaseq \
--polyg-trimming -a
```
- For better accuracy
```
gconf touch *<identifier>* --operation-mode advanced -a
```
- For modified configurations from a default pipeline, default-germline-wes
```
gconf touch *<identifier>* -s default-germline-wes \
--interval-of-interest /path/to/target-region.bed
```

### 2.2 Generate a dataset list file

The gsched command, in addition to traditional CSV editors like EXCEL, provides an efficient way to generate a CSV file listing dataset for analysis, ideal for managing large sample imports in a pipeline-compatible format.

* -s option: specifies FASTQ as the input
* -i option: sets the starting directory for browsing, here as /home/egs
* -j option: automatically assigns a job name for each sample based on the first matched group from the
    specified regular expression, such as (\w+)_R[12], found in the observed data path.
* \-\-as-csv flag: formats the output as CSV

```
gsched -s fastq -i /home/egs -j ‘(\w+)_R[12]’ --as-csv | tee data.csv
```
    
Upon execution, a text-based interface appears for file selection. Use the arrow keys and space bar to select samples. Apply the tee command to redirect the output into data.csv. Please refer to Section 0 for more information on generating sample list file.
    
![fig6](https://hackmd.io/_uploads/BkMgMF0MWe.png)
Figure 6. Example of file navigation and selection


## 3. Details of command line operation

This section provides details on each command and the available options.

### 3.1 *gconf*: Pipeline configuration settings

gconf empowers users to manage configuration parameters for data analysis pipelines. Executing this command with no additional arguments would display all available pipeline configurations. This command offers a range of subcommands to facilitate pipeline configuration management and customization.

- touch
    This subcommand is used for creating or updating configuration parameters through pipeline identifiers (ID(s)), providing flexibility to configure various options for different stages of analysis.
Using the -s or --schema option, users can initialize a new configuration based on a predefined pipeline identifier. Any changes made will be applied and saved to user’s workspace when the -a or --apply option is enabled. User-customized settings are stored into /home/$USER/.ngsaap/params.yaml.
    
- inspect
    This subcommand is utilized to furnish users with comprehensive information about a specific pipeline configuration. This functionality enables users to review the pipeline configuration settings through pipeline ID(s) for a detailed understanding of the analysis parameters.
- purge
    This subcommand provides the capability to remove configuration parameters based on the ID(s) of corresponding pipeline configuration. This functionality allows users to manage and clean up configurations that are no longer needed.

Predefined system-wide pipeline configurations are immutable and **CANNOT** be modified or deleted using touch or purge subcommand.

- default-germline-wgs-pcrfree
    general germline variant analysis settings for WGS data with PCR-free library preparation.
    
- default-germline-wgs-pcr
    general germline variant analysis settings for WGS data with PCR-based library preparation.
    
- default-germline-wes
    general germline variant analysis settings for WES data using exome capture-based enrichment, with PCR bias correction enabled.
    
- novaseq-default-germline-wgs-pcrfree specialized germline variant analysis settings for PCR-free WGS data sequenced using two-channel SBS technology, such as Illumina NovaSeq series. Poly-G trimming for read mapping is enabled.
    
- novaseq-defualt-germline-wgs-pcr
    specialized germline variant analysis settings for PCR-based WGS data sequenced using two-channel SBS technology, such as Illumina NovaSeq series. Poly-G trimming for read mapping is enabled.
    
- novaseq-defualt-germline-wes
    specialized germline variant analysis settings for WES data sequenced using two-channel SBS technology, such as Illumina NovaSeq series, and exome capture-based enrichment. PCR bias correction is enabled.

Table 1 Available options for GeneASIC NGSAAP global pipeline configuration


| Option                           | Description                                                                                                                 |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| \-\-sequencing-platform *<CHOICE>* | The option sets the sequencing platform. *<CHOICE>* is available for *mgiseq, novaseq or element*. The default is *mgiseq*. |
| \-\-sequencing-datatype *<CHOICE>* | The option sets the sequencing data type. *<CHOICE>* is available for *wgs* or *wes*. The default is *wgs*.|
| \-\-sequencing-protocol *<CHOICE>* | The option sets the library preparation protocol for the sequencing reads.*<CHOICE>* is available for *pcrfree* or *pcr*. The default is *pcr*. |
| \-\-operation-mode *<CHOICE>*   | The option sets the analysis mode. The basic mode provides the most efficient performance while the advanced mode offers higher accuracy. *<CHOICE>* is available for *basic* or *advanced*. The default is *basic*.     |
| \-\-model-path *PATH*     | The option sets the path to an external machine learning filtering model, overwriting all pipeline configurations. The default is *null*.     |
    

Table 2 Available options for GeneASIC NGSAAP read mapper


| Option | Description | 
| -------- | -------- | 
| \-\-qual-trimming     | The option trims read tails with low base quality. Apply --no-qual-trimming to disable quality trimming.     | 
| \-\-polyg-trimming     | The option trims poly-G tails from sequences. Apply --no-polyg-trimming to disable poly-G tails trimming.     | 
| \-\-adaptor-trimming *<CHOICE>*     | The option trims adaptor sequences. *<CHOICE>* is available for different trimming methods, *hardclip*, *softclip* and *none*. The default is *softclip*.     | 


Table 3 Available options for GeneASIC NGSAAP variant caller


| Option | Description |
| -------- | -------- |
| \-\-adaptive-pruning |The option enables adaptively increasing the minimum support for haplotype assemblies according to the observed region read count. Enabling this flag reduces sensitivity to sequencing errors in regions with high-coverage areas, improving the speed of calling process.|
|\-\-duplicate-removal|The option removes PCR duplication artifacts. It is not recommended to use for PCR-free samples.|
|\-\-emit-ref-confidence | The option emits reference confidence scores.|
| \-\-max-reads-per-start *<INT>* |The option controls the maximum number of reads to keep per alignment start position. Reads above this threshold would be downsampled. This value should be a positive integer. The default is *4*.|
| \-\-interval-of-interest *PATH* |The option refers to intervals of interest in BED format. It specifies the genomic regions for analysis. *PATH* should be assigned to an absolute path of the BED file for proper reference genome assembly. The default is *null*. |

Table 4 Available options for structural variant (SV) caller


| Option | Description |
| -------- | -------- |
| \-\-manta-min-candidatevariant-size *<INT>* |The option sets the minimum size for SVs and indels to be considered in candidate discovery and reporting. The default is *8*.|
| \-\-manta-min-edgeobservations *<INT>* |The option filters graph edges by removing those supported by fewer than *<INT>* observations. The default is *3*.|
|\-\-manta-graph-node-maxedge-count *<INT>*|The option limits the number of edges evaluated between nodes when both nodes have an edge count higher than the specified value. The default is *10*.|
| \-\-manta-min-candidatespanning-count *<INT>*|  The option sets the minimum number of spanning support observations required for SVs and indels to be included in candidate discovery and reporting. The default is *3*.|
|\-\-manta-min-scored-variantsize *<INT>* | The option sets the minimum size (in base pairs) for SVs and indels that should be scored and reported after candidate identification. The default is *50*.|
|\-\-manta-min-diploid-variantscore *<INT>* | The option sets the minimum QUAL score for a variant to be included in the diploid VCF. The default is *10*. |
| \-\-manta-min-pass-diploidvariant-score *<INT>* | The option sets the QUAL score threshold for marking a variant as filtered in the diploid VCF. The default is 20. |
|\-\-manta-min-pass-diploid-GTscore *<INT>* | The option sets the minimum genotype quality score for including single samples in a variant entry in the diploid VCF. The default is 15. |
|\-\-manta-min-somatic-score *<INT>* | The option sets the minimum quality score for somatic variants to be included in the somatic VCF. The default is *10*. |
| \-\-manta-min-pass-somaticscore *<INT>* | The option sets the quality score threshold for filtering somatic variants in the somatic VCF. The default is *30*. |
| \-\-manta-enable-remote-readretrieval-for-insertions-ingermline-calling-modes *<INT>* |The option enables retrieval of mate reads in remote locations with low mapping quality to improve assembly of germline insertions. Set *<INT>* as 1 to enable and 0 to disable. The default is *1*. |
|\-\-manta-enable-remote-readretrieval-for-insertions-incancer-calling-modes *<INT>* |The option enables retrieval of mate reads in remote locations with low mapping quality to improve assembly of insertions in somatic calling modes. Set *<INT>* as 1 to enable and 0 to disable. The default is 0.|
| \-\-manta-use-overlap-pairevidence *<INT>* | The option defines whether overlapping read pairs should be used as evidence. Set *<INT>* as 1 to include and 0 to exclude. The default is *0*. |
| \-\-manta-enable-evidencesignal-filter *<INT>* | The option enables a filter for candidates with insignificant evidence signals. Set *<INT>* as 1 to enable and 0 to disable. The default is *1*. |


Table 5 Available options for short tandem repeat size estimation
| Option | Description |
| ------ | ----------- |
| \-\-repeat-region-extensionlength *<INT>* |The option sets the search radius for informative reads around target regions. The default is *1000*. |
| \-\-repeat-variant-catalog *PATH* |The option sets the path to JSON catalog for variant genotyping. The default is */opt/geneasic/database/ehunter/variant_catalog.{hg}.json* |
    
Table 6 Available options for HLA typing
| Option | Description |
| ------ | ----------- |
|  \-\-hla-sample-population *<CHOICE>* | The option sets the population for HLA typing. *<CHOICE>* is available for *Asian, Black, Caucasian, Unknown*, and nonuse. The default is *Unknown*. |
| \-\-hla-exon-typing | The option enables exon typing for HLA. |
| \-\-hla-min-maf *<FLOAT>*| The option sets the minimum minor allele frequency for HLA analysis.The default is 0.1.|

Table 7 Available options for ROH
| Option | Description |
| ------ | ----------- |
|\-\-roh-min-size *<INT>* | The option sets the minimum variant size to report in million base pairs. The default is 300. |


Table 8 Available options for structural variant annotations
| Option | Description |
| ------ | ----------- |
| \-\-annotsv-sv-min-size *<INT>*| The option sets the SV size (bp) for annotation. The default is *50*. |

Table 9 Available options for variant interpretation report
| Option| Description |
| ------------------- | ----------- |
|\-\-transcript-annotation-mode *<STR> [STR …]* |The option specifies transcript annotation database. *<STR>* is available for Ensembl and Refseq. The default is *[ Ensembl ]*. |
| \-\-disease-trait *<CHOICE>* | The option specifies the disease trait for annotation. *<CHOICE>* is available for *Universal and Stroke* in germline analysis, and for *Breast,Lung, Pancreatic, Colorectal, Chronic_Myelogenous_Leukemia, Cholangiocarcinoma* in somatic analysis. The default is Universal. |
|\-\-annotate-all-disease-related-genes| The option annotates all genes related to disease. An *.xlsx* file named *<jbname>*_all_annotation.csv will be generated once the flag is enabled. Only pathogenic variants are annotated by *default*. |
|\-\-disease-related-gene-list *PATH* |The option specifies the path to the assigned gene list, allowing the analysis to focus on disease-related genes. When applying with the default value *null*, */opt/geneasic/tertiary/<assembly>/GeneList/omimGene_list.txt* is referenced. The default is null. |
|\-\-secondary-finding-gene-list *PATH* |The option specifies the path to the secondary finding gene list. When applying with the default value null, */opt/geneasic/tertiary/<assembly>/GeneList/secondaryFinding.txt* is referenced. The default is *null*. |
|\-\-carrier-screening-gene-list *PATH*| The option specifies the path to the carrier screening gene list. When applying with the default value null, */opt/geneasic/tertiary/<assembly>/GeneList/carrierScreening.txt* is referenced. The default is *null*. |
| \-\-clonal-gene-list *PATH* | The option specifies the path to the clonal hematopoiesis gene list. When applying with the default value null, */opt/geneasic/tertiary/<assembly>/GeneList/clonalGene_list.txt* is referenced.The default is *null*. |
| \-\-inhouse-snv-db *PATH* |The option specifies the path to the aggregative in-house database used for SNV filtering. The default is *null*. |
|\-\-inhouse-snv-cutoff *<FLOAT>* |The option sets the SNV filtering threshold, where the value should be between 0 and 1. The default is *0.2*. |
|\-\-inhouse-sv-db *PATH* | The option specifies the path to the aggregative in-house database used for SV filtering. The default is *null*. |
| \-\-inhouse-sv-cutoff *<FLOAT>* |The option sets the SV filtering threshold, where the value should be between 0 and 1. The default is *0.2*.  |

### 3.2 gsched: Sample data configuration

The gsched command facilitates the creation of a YAML file for genomic data analysis. It enables users to
configure analysis, select import data, assign output and generate task lists in .yaml format.

An example of generating a task list, batch.yaml, with a CSV file:
```
#generate a task list with a CSV file
gsched -b batch.yaml \
-p default-germline-wgs-pcrfree -s fastq -t bam vcf \
-i /home/egs - f data.csv -o results -a
```
    
- **Configure Analysis**
    Allows option -g to select a reference genome version and option -p to assign a pipeline configuration
    
- **Import Data**
    Specifies import data directories and/or formats for analysis
    
- **Assign Output**
    Specifies required analysis output and assigns an output destination
    
- **Generate Task Lists**
    Creates YAML files with all configurations
    

Table 10 Available options for analysis configuration
| Option | Description |
| ------ | ----------- |
| -g, \-\-assembly *<VERSION>* |  The option controls the reference genome for genome assembly. *<VERSION>* should be specified as hs38DH, hg38, hg19 or hs37d5. Thedefault is hs38d2.<br>• hs38DH<br>Corresponds to hs38DH and features an adapted masked approach to handle native reference ALT contigs. Strategic regions are masked to increased accuracy.<br>• hg38<br>hg38 is the latest reference, corresponding to hs38DH. It features an adapted masked approach specifically designed to handle and incorporate the native reference ALT contigs (Alternate Loci). Strategic regions are masked to significantly increase mapping accuracy and better represent genomic diversity.<br>• hs37d5<br>Derived from the "GRCh37" human genome reference. This assembly includes decoy sequences for improved mapping and variant calling.<br>• hg19<br> hg19 is the preceding reference. It lacks the dedicated ALT contigs and the corresponding specialized adapted masking features of hg38. Its masking strategy is simpler, which results in lower mapping accuracy and less complete representation in highly diverse|
| -p, \-\-pipeline *<ID>* | The option specifies a pipeline configuration for analysis. To see available options, simply press the TAB key twice after typing *-p or \-\-pipeline*. Alternatively, *gconf insepct* command can show all the available configurations as well.|

Table 11 Available options for data import
| Option | Description |
| ------ | ----------- |
| -s, \-\-source *<FORMAT>* | The option specifies input sources by data format. <FORMAT> can be one or more choices as followings.<br>• fastq<br> Sets FASTQ as input for paired-end sequencing data. Files should be stored separately, using extensions like 1.fastq.gz, 1.fq.gz, or 1_001.fastq.gz for read segment 1, and 2.fastq.gz, 2.fq.gz, or 2_001.fastq.gz for read segment 2.<br>• bam, vcf<br>Specifies analysis-ready BAM and VCF files as input. Files should end with .bam and .vcf, respectively, with optional index files (.bai or .tbi).<br>• sv<br>Specifies an analysis-ready SV file ending in .sv.vcf.gz as input, with an optional .tbi index file.<br>• px<br>Loads sample information from a .px.json file. The schema includes the following fields, all of which are blank by default.<br>• Order Number: Unique identifier for the order.<br>• Client Order Number: Identifier assigned by the client for tracking.<br>• Ordering Physician: Name of the physician who ordered the test.<br>• Physician ID: Unique identifier for the physician.<br>• Patient Name: Full name of the patient.<br>• Patient ID: Unique identifier for the patient.<br>• Birthday: Date of birth of the patient.<br>• Sex: Gender of the patient.<br>• Age: Age of the patient at the time of sample collection.<br>• Sample Type: Type of sample collected (e.g., blood, tissue).<br>• Sample Source: Origin of the sample (e.g., specific anatomical site).<br>• Collect Date: Date when the sample was collected.<br>• Sequencing Type: Type of sequencing performed (e.g., WGS, WES).<br>• Symptom: Relevant symptoms or conditions associated with thepatient.<br>• Note: Additional remarks or instructions related to the sample. |
| -i, \-\-import-dir *<DIR>* | The option enables files navigation in a designated directory to select data for analysis. Use the arrow keys to move through files and directories, press the space bar to select files of interest, press ENTER to stop navigation and export the selection. |
|-j, \-\-jbname *<REGEX>*|The option assigns grouping labels based on a specific regular expression pattern that matches part of file absolute path. By default, *<REGEX>* uses the pattern *([\^\\/]+)\\/[^\\/]+$*, which assigns the parent directory name as the job name, ideal for setups where each sample resides in its own folder. This default pattern simplifies job naming and grouping, allowing efficient data organization by directory structure.|
| -x, \-\-exclude *<DIR>*|The option excludes unrelated files or directories from file navigation enabled by *-s* option which refines the list of candidates for selection. |
| \-\-as-csv | The option generates a CSV sample list along with the *-i* and *-s* options, and the optional *-j* option. Apply the *tee* command to redirect the standard output to files with the extensions *.csv*.  |
| \-\-as-tsv | The option generates a TSV sample list along with the *-i* and *-s* options, and the optional *-j* option. Apply the tee command to redirect the standard output to files with the extensions *.tsv*.|
| -f, \-\-files-from *<DIR>* |The option imports data directly from CSV or TSV sample list. The sample list must adhere to the following fields: <br><jbname>,<source-type>/<index>,<source-path>,<suffix-size><br>• jbname: Job name for the specific input data and subsequent analysis<br>• source-type: Allowed choices specified in the -s option<br>• index: Specific assignment for the corresponding source type<br>• source-path: Location of the input files<br>• suffix-size: An optional field generated by gsched for resolving ambiguity<br><br>A CSV or TSV sample list can be generated by option *\-\-as-csv* or *\-\-as-tsv* along with the *-i* and *-s* options, and the optional *-j* option.|
    

To accurately extract a group label, users can test their regular expression using online tools such as regex101.com.
    
In Figure 7 , the expression (\w+) captures sample1, sample2, and sample3 from test strings (highlighted in green) for each sample. Be sure to select “Python” with the FLAVOR option when testing. To preview grouping  results before finalizing, enable the --as-csv or --as-tsv flag when running the gsched command. This preview step helps ensure accuracy and clarity in job name assignments before proceeding.
    

![fig7](https://hackmd.io/_uploads/SyHRxnCGZx.png)

Figure 7. Example of testing group label with online regular expression tester

Table 12 Available options for result output
|  Option  | Description         |
| ----------- | ---------------------------------- |
| -t, \-\-target *<CHOICE>*  |  The option specifies the required analysis and items for output results. *<CHOICE>* can be one or more choices as followings.<br>fastq<br>Creates a symbolic link to the source FASTQ file.<br>• bam<br>Performs short-read alignment using the GeneASIC NGSAAP mapper, generating a BAM file and an index file with the extension .bai. When specified with the -s option and an existing BAM file is detected, this option creates a symbolic link to the source BAM file.<br>• vcf<br>Performs GATK-like short-variant calling using GeneASIC NGSAAP caller, producing a VCF file ending with the extension *.vcf.gz* as well as an index file with the *.vcf.gz*.tbi extension. When specified with the *-s* option and an existing VCF file is detected, this option creates a symbolic link to the source VCF file.<br>• sv<br>Invokes Manta to discover structural variants, generating a call file with the *.sv.vcf.gz* extension and a corresponding *.sv.vcf.gz.tbi* file. If an existing .sv.vcf.gz file is detected when specified with the -s option, this option creates a symbolic link to the source of input file instead.<br>• px<br>Creates a symbolic link to the source input .px.json file.<br>• cnv<br>Performs copy number variant analysis using the ASCAT R package, followed by the SLMSuite for segmentation. Enabling this option generates a call file with the extension *.segment.SLMed.txt.*<br>• upd<br>Performs potential uniparental disomy extraction, generating a list of candidates with the extension *.upd.txt.* Results are used for variant interpretation.<br>• smn<br>Performs copy number variant analysis specific to SMN1, SMN2, as well as SMN2Δ7-8 (SMN2 with a deletion of Exon7-8), producing an output file with the extension *.smn.txt.*<br>• roh<br>Performs runs of homozygosity analysis using GeneASIC ROH caller,generating an output file with the extension *.roh.bed.*<br>• str<br>Targets short tandem repeat size estimation using ExpansionHunter. The default variant catalog is located at */opt/geneasic/config/<assembly>_variant_catalog.json*, where the assembly can be either hs38d2 or hs37d5. Results are stored in a *.repeat.vcf* file.<br>• hla<br>Targets human leukocyte antigen typing using SpecHLA. The summary is exported in a *.hla.txt file*.<br>• mito<br>Conducts mitochondrial variant calling using GATK Mutect2 in mitochondrial mode, followed by GATK FilterMutectCalls. The output files include a *.mitochondria.filtered.vcf.gz* file and its accompanying index file, which are exported to the specified output destination.<br>• ch<br>Performs clonal hematopoiesis analysis using GATK Mutect2 with best practices for somatic variant calling in tumor-only mode. This workflow includes GATK GetPileupSummaries, CalculateContamination, and FilterMutectCalls, followed by a hard filter for confident germline variants. Results are stored in a *.ch.filtered.clean.vcf.gz file*, along with its index file, and a *.stats* file.<br>• holmes<br>Conducts automated clinical germline variant annotation and classification using GeneASIC Holmes, producing an output file in *.tsv.gz* format.<br>• holmes_vcf<br> Attaches annotations and interpretations processed by GeneASIC Holmes to s VCF file, resulting with the extension *.holmes.vcf.gz.* This format is compatible with JBrowse 2 for rendering the CSQ field, which is used by Ensembl VEP.<br>• annotsv<br>Conducts structural variant annotations and ranking, generating an output file with the extension *.sv.annotated.tsv*.<br>• report<br>Performs variant interpretation on the called variants among diseaserelated genes, ACMG-recommended secondary findings, and carrier screening gene regions using ANNOVAR, along with gnomAD, ClinVar, CADD, Revel, and GWAS databases. It also includes additional pharmacogenomics clinical annotation via PharmCAT, and scoring polygenic risk through PGS catalog. Results are aggregated into a PDF and a XLSX file, with an optional _all_annotations.csv generated if the *\-\-annotate-all-disease-related-genes* flag is specified in gconf command.<br>• report_csv<br>Exports all CSV files generated from variant interpretation.  |
| -T, \-\-default-targets *<CHOICE>* | To simplify the process of specifying analysis and output items with option *-t*, the option *-T* allows user to select multiple choices with a single input. *<CHOICE>* is available for:<br>• germline-basic<br>Specifies bam, vcf, holmes, and report as targets<br>germline-full<br>Specifies *bam, vcf, cnv, upd, smn, str, sv, hla, roh, mito, ch, holmes, annotsv,report* as targets |
| -o, \-\-export-dir <DIR>  |   The option specifies the output directory. The default is current directory if no directory specified. |
|  \-\-group-by-jbname  |    The option organizes results into separate folders based on the job name.   |
|  \-\-flatten-directory-structure  | The option flattens nested directories, saving all output files in a single directory.  |

Table 13 Available options for task list generation
| Option | Description |
| ------ | ----------- |
| -b *PATH* | The option specifies the path to the batch configuration file in YAML format. If no path is provided, a task list will be stored as batch.yaml under the current directory. A YAML file can be overwritten multiple times to configure samples with different pipeline settings within a single file. |
| \-\-jbname-suffix | The option appends a suffix string to the collided job name to resolve any ambiguity related to job name collisions. |
|-a, \-\-apply | The option allows exiting preview mode and saves the configuration. |
| \-\-overwrite | The option replaces existing job names. |
| \-\-skip |  The option bypasses jobs with name collisions. |


### 3.3 gsub: Job submission

The gsub command allows users to submit tasks specified in the batch.yaml file, created using the gsched command, to the GeneASIC NGSAAP job queue. To initiate the submission of sample analysis tasks, execute the following command:

```
gsub batch.yaml
```

Upon execution, this command would provide a unique batch ID for your submission, create the corresponding files and directories defined in the batch.yaml file, and generate log files in the current working directory. To monitor the job status and accessing detailed runtime information, please refer to Section 0 for guidance on using the *gwatch* command.

Table 14 Available job submission options
| Option | Description |
| ------ | ----------- |
| -a, \-\-array *<STR>* | The option allows users to select samples of interest for submission using 1-based array index values which the array index starts from 1. The option argument can be in the form of comma-separated values, a contiguous range of index values, and an optional step size, as shown in the example below:<br>• -a 1-10<br>Submits a job array with index values between 1 and 10<br>• -a 1,3,5,7<br>Submits a job array with index values of 1, 3, 5 and 7<br>• -a 1-7:2 Submits a job array with index values between 1 and 7 with a step size of 2 The default setting make all tasks defined in batch.yaml submitted to the GeneASIC NGSAAP job queue. |
| -w, \-\-nodelist *<STR>* | The option allows users to specify a specific list of computing hosts in a commaseparated string format. The analysis job will be scheduled on these specified hosts as needed to satisfy the resource requirements. Any duplicate node names in the list will be ignored, ensuring that each node is counted only once. |
|-x, \-\-exclude *<STR>* | The option allows users to specify a list of hosts that should be excluded from the scheduling process. It ensures that certain nodes are not used for jobs, allowing for more control over the resource allocation in the analysis pipeline. |

### 3.4 gwatch: Job queue viewer

The gwatch command empowers users to effortlessly monitor job status and access detailed runtime information within the system. By executing the gwatch command without any additional arguments, users can launch the text-based job status viewer.

Within the viewer, each execution record is categorized as either “In Progress” or “Finished.” Users can choose to view these records in either a concise or verbose mode and conveniently switch between the two using arrow keys and space bar.

In concise mode, each record is uniquely identified by a Job ID presented as “BATCHID_TASKID.” Additionally, each task is associated with a job name, parsed by the --gsched command, and marked with one of the following states to indicate its progress:

- RUNNING: Indicates tasks that are actively being executed.
- PENDING: Denotes tasks waiting in queue for available resources to commence.
- COMPLETE: Represents tasks that have successfully finished their execution.
- FAILED: Marks tasks that encountered errors or issues during execution and did not complete as intended.
- CANCELLED: Identifies tasks intentionally terminated before completion.

Furthermore, each record in concise mode displays the job’s elapsed time, providing users with a quick overview of task execution durations.

In verbose mode, users can access not only the information about job ownership, computing node name, submission, start and end timepoints, but also the runtime information at pipelining stage level.

By default, gwatch retrieves job execution history within a 4 - hour window. However, users can specify the desired history window using the --within option, such as --within 8 to review the job execution history in the
past 8 hours.

### 3.5 gcancel : Job cancellation

The gcancel command is used to terminate the jobs that are under control of GeneASIC NGSAAP.

Table 15 Available options for job termination
| Option | Description |
| ------ | ----------- |
| \-\-batch *<ID> [ID ...]* | The option allows users to terminate a batch of tasks by specifying one or more batch ID(s). |
| \-\-jobid *<ID> [ID ...]* |The option allows users to terminate the task(s) by specifying one or more job ID(s). |
|\-\-user *<STR>* | The option restricts cancel operation to jobs owned by specific user. |
| \-\-me | The option restricts cancel operation to jobs owned by current user. |


### 3.6 gversion : Program Version Viewer

The gversion command displays comprehensive version information for GeneASIC NGSAAP including hardware, firmware, command line tools, the third party libraries, and genomic database versions.
![fig8](https://hackmd.io/_uploads/HJbCIK1QWl.png)

Figure 8. Example of GeneASIC NGSAAP version information


### 3.7 acctmgr : User Account Manager

The GeneASIC Account Manager (acctmgr) is a command-line utility for managing user accounts and groups within the NGSAAP environment.
    
⚠ **Permissions & Safety**
Before using these commands, please ensure you have the necessary administrative permissions.
**Warning:** Deleting users or groups is an **irreversible** action. Please proceed with extreme caution.

**Command Reference**

- cctmgr list
Lists all existing user accounts and groups in the system, providing a complete overview of the current
configuration.
- acctmgr +g <groupname> [<gid>]
Creates a new user group. If the optional group ID (<gid>) is not specified, the system will assign one
automatically.
- acctmgr +u <username> <groupname> [<uid>]
Adds a new user to a primary group. An optional user ID (<uid>) can be provided; otherwise, one will be
assigned automatically. After creation, you will be prompted to set the new user's password.
- acctmgr +G <username> <groupname>
Adds an existing user to a secondary group. This is typically used to grant additional permissions.
o **Example:** acctmgr +G john fpgauser adds user john to the fpgauser group, granting them access to analyze
genomic data using the FPGA.
- acctmgr -u <username>
Permanently deletes a user from the system.
- acctmgr -g <groupname>
Permanently deletes a user group from the system.
- acctmgr -G <username> <groupname>
Removes a user from a specified secondary group without deleting the user's account.


## 4. Revision History
| Version | Date | Changes |
| ------ | --- | ----------- |
| 1.1.0_b | 2025 Dec | Minor update |
| 1.1.0_a| 2024 Nov |  Update the full features of GeneASIC NGSAAP All-in-One version  |
| 1.0.0  | 2023 Mar|  Initial version   |