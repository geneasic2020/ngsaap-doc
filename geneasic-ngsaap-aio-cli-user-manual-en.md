---
title: GeneASIC NGSAAP All-in-One Command Line Tools User Manual

---

```
Contact information:
GeneASIC Technologies Corp.
info@geneasic.com
```
Version 1.1.0_c

# GeneASIC NGSAAP All-in-One Command Line Tools User Manual

## Before reading the User Manual
This user manual is designated for GeneASIC NGSAAP All-in-One v1.1.0. Please read through this user manual before using the GeneASIC NGSAAP.

All figures included in this document serve as illustrative purpose only. The actual software and content may vary.
For precise information, please consult the specific software version you have acquired.

GeneASIC Technologies Corp. has taken diligent steps to ensure the accuracy of this manual. However, GeneASIC Technologies Corp. assumes no liability for errors or omissions and reserves the right to update both the manual and the software to improve reliability, performance, or design.

Trademarks and brand names mentioned in this manual are the property of their respective owners.

For futher inquires, please contact GeneASIC Technologies Corp.

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
   - 3.6 gviz : JBrowse wizard
   - 3.7 gversion : Program Version Viewer
   - 3.8 acctmgr : User Account Manager
- 4. Revision History


## 1. Introduction

The GeneASIC NGSAAP (Next-Generation Sequencing Analysis Acceleration Platform) is a hardware-accelerated NGS secondary analysis system powered by FPGA (Field-Programmable Gate Array) technology. Beyond high-speed genomic analysis, the platform is designed for scalability and ease of use, featuring a streaming architecture that processes data in real-time within a distributed computing framework.

### 1.1 Computing architecture

The GeneASIC NGSAAP features a distributed computing architecture optimized for processing massive genomic datasets. Each node is equipped with FPGA acceleration cards to expedite core tasks such as read mapping and variant calling. This specialized hardware significantly reduces analysis time while freeing up CPU resources for secondary tasks, including advanced variant discovery, database annotation, and clinical significance interpretation.
![1](https://hackmd.io/_uploads/S1YeYlRr-x.png)


Figure 1. GeneASIC NGSAAP computing architecture

Resource allocation and task management are seamlessly coordinated by SLURM, Snakemake, and the GeneASIC pipelining controller, ensuring scalability for projects of any size. SLURM manages job queues and automatically dispatches tasks as nodes become available. Integrated with Snakemake, the controller optimizes the utilization of FPGA, CPU, memory, and disk resources, efficiently managing complex workloads even on standard workstations. 

The system supports multi-user environments, allowing concurrent job submissions without manual scheduling concerns. These features significantly enhance GeneASIC NGSAAP’s usability for large-scale projects, maximizing capacity and efficiency in genomic data analysis.

The data flow allows for direct input streaming from sources such as USB drives, MicroSD cards, network-attached storage (NAS) or data centers with network file systems (NFS) support. This enables simultaneous data transfer and processing. Results are securely transferred to user-specific storage upon completion. All operations are conducted on-premises within a local area network (LAN) to ensure maximum data privacy and eliminate the need to prefetch large datasets (e.g. FASTQ files).

In summary, GeneASIC NGSAAP leverages FPGAs to accelerate core analyses and utilizes SLURM for efficient, comprehensive, secure, and scalable genomic data analysis. Multi-user support and streamlined data flow make GeneASIC NGSAAP a powerful tool for accelerating genomic discoveries without computational bottlenecks.

### 1.2 Genomic variant analysis workflow

The FPGA-powered pipeline consists of two primary stages for short variant analysis: read mapping and variant calling, followed by comprehensive variant interpretation (e.g. structural variant analysis, variant annotation etc) in the All-in-One version.

**Stage 1. Read Mapping**

The process begins by mapping raw sequencing reads to a reference genome. This stage includes crucial preprocessing steps, including read trimming to eliminate low-quality bases or adaptors, the organization of reads by their positional coordinates. 

**Stage 2. Variant Calling**

The second stage executes short variant calling accelerated by FPGA hardware. Machine learning refinement is then applied to filter the identified variants. PCR-bias correction is optionally applied, depending on library preparation settings. The output includes a detailed catalog of SNPs and INDELs in VCF format. Additionally, in the All-in-One version, this system performs CPU-powered analyses, as well as structural variant calling (Manta), copy number analysis (ASCAT, SLM and SMNCopyNumberCaller), potential uniparental disomy extraction, short tandem repeat size estimation (ExpansionHunter), HLA typing (SpecHLA), clonal hematopoiesis analysis and mitochondrial variant calling via GATK Mutect2.

**Stage 3. Variant Interpretation**

In the final stage, variants are annotated and classified according to ACMP-AMP guidelines. Structural variants are interpreted using AnnotSV. This stage also covers carrier screening and secondary findings recommended by ACMG; pharmacogenomics annotation, and runs of homozygosity (ROH) analysis in this stage. All results are finally consolidated into reports in XLSX and PDF formats.

![3](https://hackmd.io/_uploads/HJ9a5WCSZe.png)

Figure 2. GeneASIC NGSAAP workflow


## 2. Example of WGS analysis

This section provides a step-by-step workflow of a standard Whole Genome Sequencing(WGS) analysis.

### Step 1. Prepare workspaces and datasets
```
#creat a working directory
mkdir example
cd example
```

For batch analysis, we recommend creating a CSV file that specifies job names to their respective FASTQ file path. Below is an example of a data.csv file structure:

```
sample1,fastq/0,/home/egs/sample1_R1.fastq.gz
sample1,fastq/1,/home/egs/sample1_R2.fastq.gz
sample2,fastq/0,/home/egs/sample2_R1.fastq.gz
sample2,fastq/1,/home/egs/sample2_R2.fastq.gz
```
fastq/0 and fastq/1 represent Read 1 and Read 2 of the paired-end sequencing data, respectively. Ensure that the data.csv is stored within an accessible dataset workspace.

### Step 2. Generate a task list
```
#generate a task list with a CSV file
gsched -b batch.yaml\
-p default-germline-wgs-pcrfree -s fastq -t bam vcf\
-i /home/egs -f data.csv -o results -a
```
This command generates a task list file named batch.yaml file that associates input data specified in data.csv with the pipeline configuration settings named `default-germline-wgs-pcr-free.` The `-s` option indicates that the input source is FASTQ data, while the `-t` option specifies that the outputs are BAM and VCF file. The `-i` option sets the import directory for the data source, and the `-o` defines `results` as the output directory. Apply all the settings with the `-a` option exports the batch.yaml file.

![4](https://hackmd.io/_uploads/ry2t3ZAS-g.png)

Figure 3. Example of batch.yaml content


### Step 3. Submit a task list to the job queue
```
gsub batch.yaml
```
`gsub` allows to submit the generated task lists to the GeneASIC NGSAAP job queue. Upon a successful submission, the system will return a Batch ID for tracking.

### Step 4. View job status

```
gwatch
```

`gwatch` allows to monitor the progress of job status. This command opens an interative job queue viewer as shown in Figure 4. Use the arrow keys and the ENTER to inspect specific task execution details. Press ESC to exit the viewer.
![6](https://hackmd.io/_uploads/BJ_8CbArbe.png)

Figure 4. GeneASIC NGSAAP execution details

### Step 5. Inspect results

Once the analysis is complete, navigate to the output directory specified in Step 2 to review your results.
![8,9](https://hackmd.io/_uploads/H1LoJMCSZl.png)

Figure 5. Example of output results

### 2.1 Customize pipeline configuration settings

Pipeline settings should be optimized based on the sequencing platform, library preparation, and the desired balance between speed and accuracy. Use `gconf` to create custom profiles.

Replacing *<identifier>* with an actual identifier helps customize the analysis settings to meet specific sequencing settings. The `-a` option in `gconf` saves the configuration, allowing it to be referenced in `gsched` via the `-p` option. Please refer to section 3.1 for detailed `gconf` documentation.

- For PCR-based WGS data
```
gconf touch <identifier> --sequencing-protocol pcr -a
```
- For Whole exome sequencing (WES) data
```
gconf touch <identifier> --sequencing-datetype wes \
--sequencing-protocol pcr --interval-of-interest \
/opt/geneasic/share/intervals/grch38/<target-region>.bed -a
```
- For Illumina NovaSeq or other platforms using 2-channel SBS technology
```
gconf touch <identifier> --sequencing-platform novaseq \
--polyg-trimming -a
```
- For High-Accuracy Mode
```
gconf touch *<identifier>* --operation-mode advanced -a
```
- Modifying an existing schema (e.g. default-germline-wes)
```
gconf touch *<identifier>* -s default-germline-wes \
--interval-of-interest /path/to/target-region.bed
```

### 2.2 Generate a dataset list file

Beyond using spreadsheet editors like Excel, `gsched` offers an interactive way to generate a CSV file, which is ideal for managing large sample batches.

* `-s` : Specifies FASTQ as the input
* `-i` : Set up the starting directory for browsing (e.g. `/home/egs`)
* `-j` : Uses a regular expression (e.g. `(\w+)_R[12]` to automatically assign job names based on the file names
* `--as-csv` : Formats the terminal output as a CSV string
```
gsched -s fastq -i /home/egs -j ‘(\w+)_R[12]’ --as-csv | tee data.csv
```
    
Upon execution, a text-based interface is displayed for file selection. Use the arrow keys and space bar for sample selecton. `tee` redirects the output into `data.csv`. For further information, please refer to Section 3.2.
    
![11](https://hackmd.io/_uploads/HJjKVfRSZl.png)

Figure 6. Example of file navigation and selection


## 3. Details of command line operation

This section describes each command and the available options.

### 3.1 *gconf*: Pipeline configuration settings

The `gconf` utility manages the parameters for data analysis pipelines. Executing this command with no additional arguments would display all available pipeline configurations. This command offers a range of subcommands to facilitate pipeline configuration management and customization.

- touch
    This subcommand is used for creating or updating configuration parameters through pipeline identifiers (ID(s)), providing flexibility to configure various options for different stages of analysis.
Using the -s or --schema option, users can initialize a new configuration based on a predefined pipeline identifier. Any changes made will be applied and saved to user’s workspace when the -a or --apply option is enabled. User-customized settings are stored into /home/$USER/.ngsaap/params.yaml.
    
- inspect
    This subcommand is utilized to furnish users with comprehensive information about a specific pipeline configuration. This functionality enables users to review the pipeline configuration settings through pipeline ID(s) for a detailed understanding of the analysis parameters.
- purge
    This subcommand provides the capability to remove configuration parameters based on the ID(s) of corresponding pipeline configuration. This functionality allows users to manage and clean up configurations that are no longer needed.

Predefined system-wide pipeline configurations are immutable and **CANNOT** be modified or deleted using touch or purge subcommand.

- default-germline-wgs-pcrfree
    general germline variant analysis settings for PCR-free WGS data.
    
- default-germline-wgs-pcr
    general germline variant analysis settings for PCR-based WGS data.
    
- default-germline-wes
    general germline variant analysis settings for WES data using exome capture-based enrichment, with PCR bias correction enabled.
    
- novaseq-default-germline-wgs-pcrfree optimized germline variant analysis settings for Illumina NovaSeq (using 2-channel SBS technology) PCR-free WGS data. Poly-G trimming enabled.
    
- novaseq-defualt-germline-wgs-pcr
    optimized germline variant analysis settings for Illumina Novaseq (using 2-channel SBS technology) PCR-based WGS data . Poly-G trimming enabled.
    
- novaseq-defualt-germline-wes
    optimized germline variant analysis settings for Illumina Novaseq and exome capture-based enrichment (using 2-channel SBS technology) WES data. PCR bias correction is enabled.

Table 1 Available options for GeneASIC NGSAAP global pipeline configuration


| Option                           | Description                                                                                                                 |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| \-\-sequencing-platform *<CHOICE>* | Sets the sequencing platform. *<CHOICE>* includes *mgiseq, novaseq or element*.  |
| \-\-sequencing-datatype *<CHOICE>* | Sets the sequencing data type. *<CHOICE>* includes for *wgs* or *wes*. (Default: *wgs*)|
| \-\-sequencing-protocol *<CHOICE>* | Sets the library preparation protocol .*<CHOICE>* includes *pcrfree* or *pcr*. (Default: *pcr*) |
| \-\-operation-mode *<CHOICE>*   | Sets the analysis mode. Basic mode prioritizes efficiency; Advanced mode offers higher accuracy. *<CHOICE>* includes *basic* or *advanced*. (Default: *basic*).     |
| \-\-model-path *<PATH*>     | Overwrites all pipeline configurations with a path to an external machine learning filtering model. (Default: *null*).     |
    

Table 2 Read mapper options


| Option | Description | 
| -------- | -------- | 
| \-\-qual-trimming     | Trims read tails with low base quality. Use --no-qual-trimming to disable.     | 
| \-\-polyg-trimming     | Trims poly-G tails from sequences. Use --no-polyg-trimming to disable.     | 
| \-\-adaptor-trimming *<CHOICE>*     | Sets the adaptor trimming method: hardclip, softclip or none. *<CHOICE>* includes *hardclip*, *softclip* and *none*. (Default: *softclip*）     | 


Table 3 Variant caller options


| Option | Description |
| -------- | -------- |
| \-\-adaptive-pruning |Dynamically increases the minimum support for haplotype assemblies according to the region coverage. This improves speed and reduces sensitivity to sequencing errors in high-depth areas. |
|\-\-duplicate-removal|Removes PCR duplication artifacts. Note: Not recommended for PCR-free samples.|
|\-\-emit-ref-confidence | Emits reference confidence scores.|
| \-\-max-reads-per-start *<INT>* |Sets the maximum read per alignment start position. Excess reads are downsampled. This value should be a positive integer. (Default: *4*)|
| \-\-interval-of-interest *<PATH><PATH*> |Specifies a BED format file defining specific genomic regions for analysis. Requires an absolute path *PATH* for proper reference genome assembly. (Default: *null*) |
| \-\-database-annotation  *<PATH><PATH*> |Specifies paths to variant databases for reporting metrics. (Default: *[]*) |

Table 4 Available options for structural variant (SV) caller


| Option | Description |
| -------- | -------- |
| \-\-manta-min-candidatevariant-size *<INT>* |Sets the minimum size for SVs and indels to be considered during candidate discovery and reporting. (Default: *8*)|
| \-\-manta-min-edge-observations *<INT>* |Filters graph edges by removing those supported by fewer than *<INT>* observations. (Default is *3*)|
|\-\-manta-graph-node-maxedge-count *<INT>*|Limits the number of edges evaluated between nodes when both nodes have an edge count higher than this value. (Default: *10*)|
| \-\-manta-min-candidate-spanning-count *<INT>*| Sets the minimum number of spanning support observations required for SVs and indels to be included in candidate discovery and reporting. (Default: *3*)|
|\-\-manta-min-scored-variant-size *<INT>* | Sets the minimum size (in base pairs, bp) for SVs and indels that should be scored and reported after candidate identification. (Default: *50*)|
|\-\-manta-min-diploid-variant-score *<INT>* | Sets the minimum QUAL score for a variant to be included in the diploid VCF. (Default: *10*) |
| \-\-manta-min-pass-diploid-variant-score *<INT>* | Sets the QUAL score threshold for marking a variant as filtered in the diploid VCF. (Default: *20*) |
|\-\-manta-min-pass-diploid-GT-score *<INT>* | Sets the minimum Genotype Quality (GQ) score for including single samples in a variant entry in the diploid VCF. (Default: *15*) |
|\-\-manta-min-somatic-score *<INT>* | Sets the minimum quality score for somatic variants to be included in the somatic VCF. (Default: *10*) |
| \-\-manta-min-pass-somatic-score *<INT>* | Sets the quality score threshold for filtering somatic variants in the somatic VCF. (Default: *30*) |
| \-\-manta-enable-remote-read-retrieval-for-insertions-in-germline-calling-modes *<INT>* |Enables retrieval of mate reads in remote locations with low mapping quality to improve assembly of germline insertions. Set *1* to enable, *0* to disable. (Default: *1*) |
|\-\-manta-enable-remote-read-retrieval-for-insertions-in-cancer-calling-modes *<INT>* |Enables retrieval of mate reads in remote locations to improve insertion assembly in somatic calling modes. Set *1* to enable, 0 to disable. (Default: 0)|
| \-\-manta-use-overlap-pair-evidence *<INT>* | Defines whether overlapping read pairs should be used as evidence. Set *1* to include, *0* to exclude. (Default: *0*) |
| \-\-manta-enable-evidence-signal-filter *<INT>* |Enables a filter for candidates with insignificant evidence signals. Set 1 to enable, 0 to disable. (Default: *1*) |


Table 5 Available options for short tandem repeat size estimation
| Option | Description |
| ------ | ----------- |
| \-\-repeat-region-extension-length *<INT>* |Sets the search radius(bp) for informative reads around target regions. (Default: *1000*) |
| \-\-repeat-variant-catalog *<PATH><PATH*> |Specifies the path to JSON catalog used for variant genotyping. (Default: */opt/geneasic/database/ehunter/variant_catalog.{hg}.json*) |
    
Table 6 Available options for HLA typing
| Option | Description |
| ------ | ----------- |
|  \-\-hla-sample-population *<CHOICE>* | Sets the target population for HLA typing. *<CHOICE>* includes *Asian, Black, Caucasian, Unknown*, and nonuse. (Default: *Unknown*) |
| \-\-hla-exon-typing | Enables exon-level typing for HLA. |
| \-\-hla-min-maf *<FLOAT>*| Sets the minimum minor allele frequency (MAF) for HLA analysis.(Default: *0.1*)|

Table 7 Available options for ROH
| Option | Description |
| ------ | ----------- |
|\-\-roh-min-size *<INT>* | Sets the minimum variant size to report (in million base pairs/Mbp). (Default: *300*) |


Table 8 Available options for structural variant annotations
| Option | Description |
| ------ | ----------- |
| \-\-annotsv-sv-min-size *<INT>*| Sets the minimum SV size (bp) required for annotation. (Default: *50*) |

Table 9 Available options for variant interpretation report
| Option| Description |
| ------------------- | ----------- |
|\-\-transcript-annotation-mode *<STR> [STR …]* |Specifies the transcript annotation database. *<STR>* includes Ensembl and Refseq. (Default is *Ensembl*) |
| \-\-disease-trait *<CHOICE>* | Specifies the disease trait for annotation. *Germline* includes *Universal* and *Stroke* in germline analysis, while somatic analysis includes *Breast*,*Lung*, *Pancreatic*, *Colorectal*, *Chronic_Myelogenous_Leukemia* and  *Cholangiocarcinoma*. (Default: *Universal*) |
|\-\-annotate-all-disease-related-genes| Annotates all disease-related genes. Generates an *.xlsx* file named *<jbname>*_all_annotation.csv. By *default*, only pathogenic variants are annotated. |
|\-\-disease-related-gene-list *PATH* |Specifies a custom gene list for analysis. If *null*, uses system default: */opt/geneasic/tertiary/<assembly>/GeneList/omimGene_list.txt* |
|\-\-secondary-finding-gene-list *PATH* |Specifies a custom secondary findings gene list. If *null*, uses system default: */opt/geneasic/tertiary/<assembly>/GeneList/secondaryFinding.txt* |
|\-\-carrier-screening-gene-list *PATH*| Specifies a custom carrier screening gene list. If *null*, uses */opt/geneasic/tertiary/<assembly>/GeneList/carrierScreening.txt* |
| \-\-clonal-gene-list *PATH* | Specifies a custom clonal hematopoiesis gene list. If *null*, uses */opt/geneasic/tertiary/<assembly>/GeneList/clonalGene_list.txt* |
| \-\-inhouse-snv-db *PATH* |Path to the aggregated in-house database used for SNV filtering. (Default: *null*) |
|\-\-inhouse-snv-cutoff *<FLOAT>* |Sets the SNV filtering threshold, where the value should be between 0 and 1. The default is *0.2*. |
|\-\-inhouse-sv-db *PATH* | Path to the aggregated in-house database used for SV filtering. (Default: *null*) |
| \-\-inhouse-sv-cutoff *<FLOAT>* |Sets the SV filtering threshold, where the value should be between 0 and 1. (Default is *0.2*)  |

### 3.2 gsched: Sample data configuration

The gsched command facilitates the creation of a YAML file for genomic data analysis. It enables users to
configure analysis, select import data, assign output and assign output destinations.

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
    

Table 10.1 Available options for analysis configuration
| Option | Description |
| ------ | ----------- |
| -g, \-\-assembly *<VERSION>* |  Specifies the reference genome assembly to be used. *<VERSION>* should be specified as 
    DH, hg38, hg19 or hs37d5. (Default: *hs38DH*)<br>|
| -p, \-\-pipeline *<ID>* | Specifies the pipeline configuration for the analysis. After typing *-p* or *--pipeline*, press the *TAB* key twice to view available options. You may also use the *gconf inspect* command to view configurations.|

The following table describes the genome versions available via the *-g*, *--assembly* parameter and their respective technical characteristics:
    
Table 10.2 Reference Genome Analysis Configuration Details
| Option | Description |
| ------ | ----------- |
| hs38DH (Default) |  An optimized version based on hs38DH. In addition to the full GRCh38 analysis set (including decoys and HLA), it employs an adapted masking method for ALT contigs and hard-masking for specific strategic regions. This is the highly recommended choice for Whole Genome Sequencing (WGS).https://ftp-trace.ncbi.nih.gov/1000genomes/ftp/technical/reference/GRCh38_reference_genome/GRCh38_full_analysis_set_plus_decoy_hla.fa|
| hs37d5 | The full analysis set derived from GRCh37. This version includes human decoy sequences and the EBV virus sequence to effectively "absorb" non-target fragments, reducing false-positive variant calls. It is the standard version for clinical diagnostics. https://ftp.1000genomes.ebi.ac.uk/vol1/ftp/technical/reference/phase2_reference_assembly_sequence/hs37d5.fa.gz |    
| hg19 | The UCSC standard GRCh37 reference sequence. As an earlier standard, this assembly also includes decoy sequences to improve alignment quality; it is suitable for analyses requiring comparison with historical data. https://hgdownload.soe.ucsc.edu/goldenPath/hg19/bigZips/hg19.fa.gz |
| hg38 | The UCSC standard GRCh38 reference sequence. It includes primary chromosomes and ALT contigs but lacks the human decoys provided with hs38DH. Suitable for general visualization or specific analyses where decoys are not required. https://hgdownload.soe.ucsc.edu/goldenPath/hg38/bigZips/hg38.fa.gz |

Reference genomes from different sources vary significantly in naming conventions and composition. Use the following table to quickly identify and verify your reference files:

Table 10.3 Summary of Supported Human Reference Genomes
| Option | Source |Contig Count |Prefix Style |Decoy Tag |
| ------ | ------ |----------- |----------- |----------- |
| hs38DH | NCBI |3366 |chr1, chrUn |chrUn*_decoy |
| hs37d5 | NCBI/1000 Genome |86 |1, GL* |hs37d5 |
| hg38 | UCSC |455 |chr1, chrUn |N/A |
| hg19 | UCSC |93 |chr1, chrUn |N/A |

*N/A: Not Applicable

To enhance alignment precision, high-quality analysis sets (such as hs38DH) consist of sequences with various functions. The following sections define these components:

Table 10.4 hs38DH Reference Genome Component Analysis
| Category | Contig Count |Description|
| ------ | ------ |----------- |
| Main Reference | 25 |Includes chromosomes chr1–22, chrX, chrY, and the mitochondrial genome (chrM). The core regions of the human genome. |
| Unlocalized Sequence| 42 |Labeled as chr*_random. Sequences known to belong to a specific chromosome but with unknown coordinates. |
| Unplaced Sequence |127 |Labeled as chrUn*. Sequences confirmed as human but whose source chromosome is unknown. |
| Alternative Loci | 261 |Labeled as chr*_alt. Represents highly polymorphic regions (e.g., immune genes). Used for Alt-aware alignment to prevent false positives. |
| EBV Sequence | 1 |Labeled as chrEBV. Used to absorb potential viral DNA fragments in the sample, preventing interference with human gene alignment.|
| Decoy Sequence | 2385 |Labeled as chrUn*_decoy. Acts as a "Sink" to attract repeats or non-reference fragments that cause misalignments; critical for WGS accuracy. |
| HLA Sequence |525 |Labeled as HLA-*. Allele sequences specifically designed for the complex HLA region; essential for precise HLA typing.|
    
Table 10.5 hs37d5 Reference Genome Component Analysis
| Category | Contig Count |Description|
| ------ | ------ |----------- |
| Main Reference | 25 |Includes chromosomes 1–22, X, Y, and mitochondria (MT). |
| Unlocalized Sequence| 20 |Labeled with GL numbers (GL000191.1 to GL000226.1). Known chromosome origin, unknown coordinates. |
| Unplaced Sequence |39 |Labeled with GL numbers (GL000227.1 to GL000249.1). Confirmed human, unknown source chromosome. |
| EBV Sequence | 1 |Labeled as NC_007605. Absorbs viral DNA fragments.|
| Decoy Sequence | 1 |Labeled as hs37d5. Acts as a "Sink" for repetitive/non-reference sequences to improve accuracy. |
    
Table 10.6 hg38 Reference Genome Component Analysis
| Category | Contig Count |Description|
| ------ | ------ |----------- |
| Main Reference | 25 |Primary chromosomes (chr1–22, X, Y, M). |
| Unlocalized Sequence| 42 |Labeled as chr*_random. Known chromosome, unknown coordinates. |
| Unplaced Sequence |127 |Labeled as chrUn_*. Confirmed human, unknown chromosome. |
| Alternative Loci | 261 |Labeled as chr*_alt. Used for Alt-aware alignment. |

Table 10.7 hg19 Reference Genome Component Analysis
| Category | Contig Count |Description|
| ------ | ------ |----------- |
| Main Reference | 25 |Primary chromosomes (chr1–22, X, Y, M). |
| Unlocalized Sequence| 20 |Labeled as chr*_random. Known chromosome, unknown coordinates. |
| Unplaced Sequence |39 |Labeled as chrUn_*. Confirmed human, unknown chromosome. |
| Alternative Loci | 9 |Labeled as chr*_hap?. Highly polymorphic regions used for Alt-aware alignment. |


Table 11 Available options for data import
| Option | Description |
| ------ | ----------- |
| -s, \-\-source *<FORMAT>* | Specifies input sources by data format. <FORMAT> Multiple choices can be selected: <br>• fastq<br> Sets FASTQ as input for paired-end sequencing data. Files should be stored separately, using extensions like 1.fastq.gz, 1.fq.gz, or 1_001.fastq.gz for read segment 1, and 2.fastq.gz, 2.fq.gz, or 2_001.fastq.gz for read segment 2.<br>• bam, vcf<br>Analysis-ready BAM and VCF files. Requires .bam and .vcf extensions; index files (.bai or .tbi) are optional. <br>• sv<br>Analysis-ready structural variant (SV) files ending in .sv.vcf.gz.<br>• px<br>Loads sample information from a .px.json file. The schema includes the following fields, all of which are blank by default.<br>• Order Number: Unique identifier for the order.<br>• Client Order Number: Identifier assigned by the client for tracking.<br>• Ordering Physician: Name of the physician who ordered the test.<br>• Physician ID: Unique identifier for the physician.<br>• Patient Name: Full name of the patient.<br>• Patient ID: Unique identifier for the patient.<br>• Birthday: Date of birth of the patient.<br>• Sex: Gender of the patient.<br>• Age: Age of the patient at the time of sample collection.<br>• Sample Type: Type of sample collected (e.g., blood, tissue).<br>• Sample Source: Origin of the sample (e.g., specific anatomical site).<br>• Collect Date: Date when the sample was collected.<br>• Sequencing Type: Type of sequencing performed (e.g., WGS, WES).<br>• Symptom: Relevant symptoms or conditions associated with thepatient.<br>• Note: Additional remarks or instructions related to the sample. |
| -i, \-\-import-dir *<DIR>* | Enables interactive file navigation in a designated directory. Use arrow keys to navigate, space bar to select files, and ENTER to confirm and export selections. |
|-j, \-\-jbname *<REGEX>*|Assigns grouping labels based on a specific regular expression pattern that matches part of file absolute path. By default, *<REGEX>* uses the pattern *([\^\\/]+)\\/[^\\/]+$*, which assigns the parent directory name as the job name, ideal for setups where each sample resides in its own folder. This default pattern simplifies job naming and grouping, allowing efficient data organization by directory structure.|
| -x, \-\-exclude *<DIR>*|Excludes unrelated files or directories from file navigation enabled by *-s* option which refines the list of candidates for selection. |
| \-\-as-csv | Generates a CSV sample list along with the *-i* and *-s* options, and the optional *-j* option. Apply the *tee* command to redirect the standard output to files with the extensions *.csv*.  |
| \-\-as-tsv | Generates a TSV sample list along with the *-i* and *-s* options, and the optional *-j* option. Apply the tee command to redirect the standard output to files with the extensions *.tsv*.|
| -f, \-\-files-from *<DIR>* |Imports data directly from CSV or TSV sample list. The sample list must adhere to the following fields: <br><jbname>,<source-type>/<index>,<source-path>,<suffix-size><br>• jbname: Job name for the specific input data and subsequent analysis<br>• source-type: Allowed choices specified in the -s option<br>• index: Specific assignment for the corresponding source type<br>• source-path: Location of the input files<br>• suffix-size: An optional field generated by gsched for resolving ambiguity<br><br>A CSV or TSV sample list can be generated by option *\-\-as-csv* or *\-\-as-tsv* along with the *-i* and *-s* options, and the optional *-j* option.|
    

To accurately extract a group label, users can test their regular expression using online tools such as regex101.com.
    
In Figure 7 , the expression (\w+) captures sample1, sample2, and sample3 from test strings (highlighted in green) for each sample. Be sure to select “Python” with the FLAVOR option when testing. To preview grouping  results before finalizing, enable the --as-csv or --as-tsv flag when running the gsched command. This preview step helps ensure accuracy and clarity in job name assignments before proceeding.
    
![13](https://hackmd.io/_uploads/B1Rw_i4Ibl.png)

Figure 7. Example of testing group label with online regular expression tester

Table 12 Available options for result output
|  Option  | Description         |
| ----------- | ---------------------------------- |
| -t, \-\-target *<CHOICE>*  |  Specifies the required analysis and items for output results. *<CHOICE>* can be one or more choices as followings.<br>fastq<br>Creates a symbolic link to the source FASTQ file.<br>• bam<br>Performs short-read alignment using the GeneASIC NGSAAP mapper, generating a BAM file and an index file with the extension .bai. When specified with the -s option and an existing BAM file is detected, this option creates a symbolic link to the source BAM file.<br>• vcf<br>Performs GATK-like short-variant calling using GeneASIC NGSAAP caller, producing a VCF file ending with the extension *.vcf.gz* as well as an index file with the *.vcf.gz*.tbi extension. When specified with the *-s* option and an existing VCF file is detected, this option creates a symbolic link to the source VCF file.<br>• sv<br>Invokes Manta to discover structural variants, generating a call file with the *.sv.vcf.gz* extension and a corresponding *.sv.vcf.gz.tbi* file. If an existing .sv.vcf.gz file is detected when specified with the -s option, this option creates a symbolic link to the source of input file instead.<br>• px<br>Creates a symbolic link to the source input .px.json file.<br>• cnv<br>Performs copy number variant analysis using the ASCAT R package, followed by the SLMSuite for segmentation. Enabling this option generates a call file with the extension *.segment.SLMed.txt.*<br>• upd<br>Performs potential uniparental disomy extraction, generating a list of candidates with the extension *.upd.txt.* Results are used for variant interpretation.<br>• smn<br>Performs copy number variant analysis specific to SMN1, SMN2, as well as SMN2Δ7-8 (SMN2 with a deletion of Exon7-8), producing an output file with the extension *.smn.txt.*<br>• roh<br>Performs runs of homozygosity analysis using GeneASIC ROH caller,generating an output file with the extension *.roh.bed.*<br>• str<br>Targets short tandem repeat size estimation using ExpansionHunter. The default variant catalog is located at */opt/geneasic/config/<assembly>_variant_catalog.json*, where the assembly can be either hs38DH or hs37d5. Results are stored in a *.repeat.vcf* file.<br>• hla<br>Targets human leukocyte antigen typing using SpecHLA. The summary is exported in a *.hla.txt file*.<br>• mito<br>Conducts mitochondrial variant calling using GATK Mutect2 in mitochondrial mode, followed by GATK FilterMutectCalls. The output files include a *.mitochondria.filtered.vcf.gz* file and its accompanying index file, which are exported to the specified output destination.<br>• ch<br>Performs clonal hematopoiesis analysis using GATK Mutect2 with best practices for somatic variant calling in tumor-only mode. This workflow includes GATK GetPileupSummaries, CalculateContamination, and FilterMutectCalls, followed by a hard filter for confident germline variants. Results are stored in a *.ch.filtered.clean.vcf.gz file*, along with its index file, and a *.stats* file.<br>• holmes<br>Conducts automated clinical germline variant annotation and classification using GeneASIC Holmes, producing an output file in *.tsv.gz* format.<br>• holmes_vcf<br> Attaches annotations and interpretations processed by GeneASIC Holmes to s VCF file, resulting with the extension *.holmes.vcf.gz.* This format is compatible with JBrowse 2 for rendering the CSQ field, which is used by Ensembl VEP.<br>• annotsv<br>Conducts structural variant annotations and ranking, generating an output file with the extension *.sv.annotated.tsv*.<br>• report<br>Performs variant interpretation on the called variants among diseaserelated genes, ACMG-recommended secondary findings, and carrier screening gene regions using ANNOVAR, along with gnomAD, ClinVar, CADD, Revel, and GWAS databases. It also includes additional pharmacogenomics clinical annotation via PharmCAT, and scoring polygenic risk through PGS catalog. Results are aggregated into a PDF and a XLSX file, with an optional _all_annotations.csv generated if the *\-\-annotate-all-disease-related-genes* flag is specified in gconf command.<br>• report_csv<br>Exports all CSV files generated from variant interpretation.  |
| -T, \-\-default-targets *<CHOICE>* | To simplify the process of specifying analysis and output items with option *-t*, the option *-T* allows user to select multiple choices with a single input. *<CHOICE>* is available for:<br>• germline-basic<br>Specifies bam, vcf, holmes, and report as targets<br>germline-full<br>Specifies *bam, vcf, cnv, upd, smn, str, sv, hla, roh, mito, ch, holmes, annotsv,report* as targets |
| -o, \-\-export-dir <DIR>  |   Specifies the output directory. The default is current directory if no directory specified. |
|  \-\-group-by-jbname  |    Organizes results into separate folders based on the job name.   |
|  \-\-flatten-directory-structure  | Savs all output files in a single directory.  |

Table 13 Available options for task list generation
| Option | Description |
| ------ | ----------- |
| -b *PATH* | Specifies the path to the batch configuration file in YAML format. If no path is provided, a task list will be stored as batch.yaml under the current directory. A YAML file can be overwritten multiple times to configure samples with different pipeline settings within a single file. |
| \-\-jbname-suffix | Appends a suffix string to the collided job name to resolve any ambiguity related to job name collisions. |
|-a, \-\-apply | Allows exiting preview mode and saves the configuration. |
| \-\-overwrite | Replaces existing job names. |
| \-\-skip |  Bypasses jobs with name collisions. |


### 3.3 gsub: Job submission

The gsub command submits tasks defined in batch.yaml to the GeneASIC NGSAAP job queue. To initiate tasks, execute the following command:

```
gsub batch.yaml
```

Upon execution, the system provides a unique Batch ID, creates the necessary directory structure, and generates log files in the current working directory. To monitor job status and access detailed runtime data, please refer to Section 3.4 for guidance on using the *gwatch* command.

Table 14 Available job submission options
| Option | Description |
| ------ | ----------- |
| -a, \-\-array *<STR>* | Submits specific samples using a 1-based index. Supports comma-separated values, ranges, and step sizes:<br>• -a 1-10<br>Submits a job array with index values between 1 and 10<br>• -a 1,3,5,7<br>Submits a job array with index values of 1, 3, 5 and 7<br>• -a 1-7:2 Submits a job array with index values between 1 and 7 with a step size of 2 The default setting make all tasks defined in batch.yaml submitted to the GeneASIC NGSAAP job queue. |
| -w, \-\-nodelist *<STR>* | Specifies a specific list of computing hosts in a commaseparated string format. The analysis job will be scheduled on these specified hosts as needed to satisfy the resource requirements. Any duplicate node names in the list will be ignored, ensuring that each node is counted only once. |
|-x, \-\-exclude *<STR>* | Specifies a list of hosts that should be excluded from the scheduling process. It ensures that certain nodes are not used for jobs, allowing for more control over the resource allocation in the analysis pipeline. |

### 3.4 gwatch: Job queue viewer

The gwatch command provides a text-based interface to monitor job status and runtime information. Run the command without arguments to launch the viewer.

Within the viewer, each execution record is categorized as either “In Progress” or “Finished.” Users can choose to view these records in either a concise or verbose mode and conveniently switch between the two using arrow keys and space bar.

In concise mode, each record is uniquely identified by a Job ID presented as “BATCHID_TASKID.” Additionally, each task is associated with a job name, parsed by the --gsched command, and marked with one of the following states to indicate its progress:

- RUNNING: Tasks actively being executed.
- PENDING: Tasks waiting in the queue for available resources.
- COMPLETE: Tasks that finished successfully.
- FAILED: Tasks that encountered errors and stopped.
- CANCELLED: Tasks intentionally terminated by a user.

Furthermore, each record in concise mode displays the job’s elapsed time, providing users with a quick overview of task execution durations.

In verbose mode, users can access not only the information about job ownership, computing node name, submission, start and end timepoints, but also the runtime information at pipelining stage level.

By default, gwatch retrieves job execution history within a 4 - hour window. However, users can specify the desired history window using the --within option, such as --within 8 to review the job execution history in the
past 8 hours.

### 3.5 gcancel : Job cancellation

The gcancel command is used to terminate active or pending jobs.

Table 15 Available options for job termination
| Option | Description |
| ------ | ----------- |
| \-\-batch *<ID> [ID ...]* | Terminates a batch of tasks by specifying one or more batch ID(s). |
| \-\-jobid *<ID> [ID ...]* |Terminates the task(s) by specifying one or more job ID(s). |
|\-\-user *<STR>* | Restricts cancel operation to jobs owned by specific user. |
| \-\-me | Restricts cancel operation to jobs owned by current user. |

### 3.6 gviz : JBrowse wizard

The `gviz` command is a dedicated wizard designed for the JBrowse web application, a tool used for visualizing genomic sequencing data. Users can easily import analysis results, including sequence alignment files (BAM), variant call files (VCF), and feature tracks or annotations in various formats (such as BED or GFF3)

To launch the track selection interface and choose genomic tracks for visualization, execute the following command:

```
gviz -s /nas/homes/$USER/<results> -t /path/to/jbrowse
```

In this command, use the `-s` option to specify the directory containing your analysis results and the `-t` option to designate the JBrowse output directory. Replace <results> with the actual path to your analysis data.

![16](https://hackmd.io/_uploads/Byx4hoV8Wx.png)

Figure 8. Track Selector Example

Once the */path/to/jbrowse* folder is created, you can launch a local web server using the following command:
    
```
npx serve -S -l <port> /path/to/jbrowse
```
This command creates an entry point on the specified port. Hold the Ctrl key and click the generated link to open JBrowse and explore your results.
    
![18](https://hackmd.io/_uploads/HynYnsNU-l.png)

Figure 9. JBrowse Entry Point Example

To import additional tracks for browsing, any content placed within the *jbrowse/<results>* folder can be accessed using a path relative to the JBrowse root directory.

For comprehensive operational details, please refer to the official JBrowse documentation: https://jbrowse.org/jb2/docs/user_guide/.
    
![19](https://hackmd.io/_uploads/H1g_22iNU-l.png)

Figure 10. Steps for Importing BED Files
    
### 3.7 gversion : Program Version Viewer

The `gversion` command displays comprehensive version information for GeneASIC NGSAAP, including hardware specs, firmware, command-line tools, third-party libraries, and genomic database versions.
    
![20](https://hackmd.io/_uploads/rkzJaiEL-e.png)


Figure 11. Example of GeneASIC NGSAAP version information


### 3.8 acctmgr : User Account Manager

The GeneASIC Account Manager (acctmgr) is a command-line utility for managing user accounts and groups within the NGSAAP environment.
    
⚠ **Permissions & Safety**
Before using these commands, please ensure you have the necessary administrative permissions.
**Warning:** Deleting users or groups is an **irreversible** action. Please proceed with extreme caution.

**Command Reference**

- *acctmgr list*
Lists all existing user accounts and groups in the system, providing a complete overview of the current configuration.
- *acctmgr +g <groupname> [<gid>]*
Creates a new user group. If the optional group ID (<gid>) is not specified, the system will assign oneautomatically.
- *acctmgr +u <username> <groupname> [<uid>]*
Adds a new user to a primary group. An optional user ID (<uid>) can be provided; otherwise, one will be assigned automatically. After creation, you will be prompted to set the new user's password.
- *acctmgr +G <username> <groupname>*
Adds an existing user to a secondary group. This is typically used to grant additional permissions.
 **Example:** acctmgr +G john fpgauser adds user john to the fpgauser group, granting them access to analyze
genomic data using the FPGA.
- *acctmgr -u <username>*
Permanently deletes a user from the system.
- *acctmgr -g <groupname>*
Permanently deletes a user group from the system.
- *acctmgr -G <username> <groupname>*
Removes a user from a specified secondary group without deleting the user's account.


## 4. Revision History
| Version | Date | Changes |
| ------ | --- | ----------- |
| 1.1.0_c | 2026 Jan | Major update |
| 1.1.0_b | 2025 Dec | Minor update |
| 1.1.0_a| 2024 Nov |  Update the full features of GeneASIC NGSAAP All-in-One version  |
| 1.0.0  | 2023 Mar|  Initial version   |