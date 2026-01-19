# GeneASIC NGSAAP All-in-One 命令列工具使用者手冊
適用版本：1.1.0_b

```
聯絡資訊:
GeneASIC Technologies Corp.
info@geneasic.com
```

## 閱讀使用者手冊之前

本使用者手冊適用於 GeneASIC NGSAAP All-in-One v1.1.0 版本。
在使GeneASIC NGSAAP 之前，請務必詳細閱讀本手冊。

本文件中包含的所有圖片僅作為示意範例。實際軟體內容可能略有不同。
如需準確資訊，請參閱您所取得的軟體版本。

GeneASIC Technologies Corp. 已採取措施確保本手冊的準確性。儘管如此，GeneASIC Technologies Corp. 不對手冊中的任何錯誤或遺漏承擔責任，並保留更新手冊和軟體的權利，以提升其可靠性、效能或設計。

本手冊中提及的商標和品牌名稱均為其各自所有者的財產。

如有任何進一步問題，請聯繫 GeneASIC Technologies Corp.。

© 2025 GeneASIC Technologies Corp. 保留所有權利。

## 目錄
[toc]

## 1. 簡介

GeneASIC NGSAAP（次世代定序分析加速平台）是一套整合 FPGA（現場可程式化邏輯閘陣列）硬體加速技術的次世代定序(NGS) 二級分析系統(Secondary Analysis)。**除了強大的基因組分析效能外，GeneASIC NGSAAP 平台兼具高擴展性與易用性，採用串流架構（Streaming Architecture）並於分散式運算框架中實現即時資料處理。**

### 1.1 運算架構

GeneASIC NGSAAP 採用分散式運算架構，**優化了大型基因組資料集的處理效率。每個運算節點均配置 FPGA 加速卡，能顯著提升序列比對（Read Mapping）與變異偵測（Variant Calling）等運算密集型任務的速度。此架構不僅加速了核心分析流程，更有效釋放 CPU 資源，使其可用於進階變異探索、資料庫註釋及臨床判讀等任務。**

![1](https://hackmd.io/_uploads/HkChuypNZe.png)

圖 1. GeneASIC NGSAAP 運算架構

**系統資源分配與任務排程由 SLURM、Snakemake 及 GeneASIC 流程控制器 協同運作**，為任何規模的專案提供可擴展性。**SLURM 負責作業佇列管理與節點分派；流程控制器則與 Snakemake 結合，優化 FPGA、CPU、記憶體與磁碟資源的配置。即使在消費級硬體上，亦能高效管理多樣化任務。此外，系統支援多使用者環境，允許多名使用者同時提交作業而無須擔心資源競爭，極大化大規模專案的分析吞吐量。**

**數據流方面，GeneASIC NGSAAP系統支援從 USB、MicroSD、網路附加儲存（NAS）或支援網路檔案系統（NFS）儲存環境直接串流輸入，實現並行傳輸與處理。分析完成後，結果將安全地傳輸至使用者指定的儲存位置。所有運算均在區域網路（LAN）內的本地端執行，確保基因資料的高安全性與隱私，同時降低了大型資料集（如 FASTQ 檔案）預取的開銷。**

**總結來說，GeneASIC NGSAAP 利用 FPGA 加速核心分析流程，並結合 SLURM 調度系統，實現高效、全面、安全且具擴展性的基因組數據分析。透過多使用者支援與優化的數據流，GeneASIC NGSAAP 成為加速基因組科學發現的強大工具，且能有效消除運算瓶頸。**

### 1.2 基因組變異分析流程

基於 FPGA 驅動的 GeneASIC NGSAAP 流程在短變異（Short Variant）分析中包含兩個主要階段：序列比對(Read Mapping)與變異偵測(Variant Calling)，並在 All-in-One 版本中整合了額外的分析與判讀功能，例如結構變異分析、變異註釋等。

**第 1 階段：序列比對 (Read Mapping)**

此過程始於將原始定序對比至參考基因組。此階段包含關鍵的數據前處理步驟，包括修剪讀段以消除低品質鹼基或接頭序列（adaptors），以及根據其位置座標組織定序。產出為可用於後續分析的基因序列。

**第 2 階段：變異偵測 (Variant Calling)**

第二階段主要透過 FPGA 硬體加速技術處理短變異 (Short Variant) 偵測。隨後應用機器學習技術對識別出的變異進行優化與過濾。系統會根據文庫製備 (Library Preparation) 的設定，選擇性地套用 PCR 偏差校正。輸出結果包括單核苷酸多態性(SNPs)及插入缺失(INDELs) 變異偵測格式檔(VCF)。此外，在 All-in-One 版本中，此階段還執行 CPU 驅動的分析，如結構變異偵測（Structural Variant Calling, Manta）、拷貝數分析（Copy Number Analysis, ASCAT、SLM 和 SMNCopyNumberCaller）、潛在單親二倍體(Potential Uniparental Disomy)提取、短串聯重複序列(Short Tandem Repeat)大小估計（ExpansionHunter）、HLA 分型（SpecHLA）、單株性造血幹細胞增生(Clonal Hematopoiesis)分析（GATK Mutect2）以及粒線體變異偵測（GATK Mutect2 粒線體模式）。

**第 3 階段：變異判讀 (Variant Interpretation)**

在最後階段，變異會經過註釋並遵循 ACMG-AMP 指南進行嚴格的分類分析；結構變異則使用 AnnotSV 進行判讀。此外，此階段亦會完成 ACMG 建議的帶因者篩檢 (Carrier Screening) 與次要發現 (Secondary Findings)、藥物基因組學註釋、多基因風險評分 (Polygenic Risk Score) 評估以及純合子區段 (ROH) 分析。所有結果最終將彙整為 XLSX 與 PDF 格式的報告。

![3](https://hackmd.io/_uploads/BJS76yT4-x.png)
(https://)
圖 2. GeneASIC NGSAAP 工作流程

## 2\. WGS 分析範例

以下為分析全基因組定序（WGS）數據的步驟說明。

### 步驟 1. 準備工作空間與數據集

```
#建立一個工作目錄
mkdir example
cd example
```

對於批次分析，建議建立一個 CSV 檔案來指定作業名稱及其對應的 FASTQ 路徑。以下為用於分析 CSV 的範例 `data.csv` ：

```
sample1,fastq/0,/home/egs/sample1_R1.fastq.gz
sample1,fastq/1,/home/egs/sample1_R2.fastq.gz
sample2,fastq/0,/home/egs/sample2_R1.fastq.gz
sample2,fastq/1,/home/egs/sample2_R2.fastq.gz
```

`fastq/0` 和 `fastq/1` 分別代表雙端定序數據的 R1 與 R2。請確保 `data.csv` 儲存於數據集的工作空間中。

### 步驟 2. 生成任務清單

```
#使用 CSV 檔案生成任務清單
gsched -b batch.yaml\
-p default-germline-wgs-pcrfree -s fastq -t bam vcf\
-i /home/egs -f data.csv -o results -a
```

此指令會產生一個 `batch.yaml` 檔案，旨在將 `data.csv` 中指定的輸入數據與名為 `default-germline-wgs-pcr-free` 的分析流程配置進行關聯。其中，`-s` 輸入來源為 FASTQ 數據； `-t` 選項則指定輸出為 BAM 與 VCF 檔案。此外，使用`-i` 選項設定 `data.csv` 的匯入目錄，並透過`-o` 將輸出目錄定義為 `results`。最後套用 `-a` 選項以執行所有設定，隨即匯出 `batch.yaml` 檔案。

![4](https://hackmd.io/_uploads/SJkgKXAEZe.png)


圖 3. batch.yaml 內容範例

### 步驟 3. 提交任務清單至作業佇列

```
gsub batch.yaml
```

`gsub` 允許將任務清單提交至 GeneASIC NGSAAP 作業佇列。如果作業提交成功，將回傳一個批次 ID（Batch ID）。

### 步驟 4. 檢視作業狀態

```
gwatch
```

`gwatch` 指令允許檢視所有作業狀態。執行此指令後，系統將如下所示顯示作業隊列中的任務清單。要查看執行詳情，請使用方向鍵移動並按下 ENTER 鍵以選取您感興趣的特定任務。按下 ESC 鍵即可退出作業狀態檢視介面。

![6](https://hackmd.io/_uploads/HydpT7AVZx.png)


圖 4. GeneASIC NGSAAP 執行詳情

### 步驟 5. 檢查結果

數據分析完成後，您可以在步驟 2 指定的輸出目錄中檢視結果。
![8,9](https://hackmd.io/_uploads/HyVZ4ERVZe.png)

圖 5. 輸出結果範例

### 2.1 自訂流程配置設定

合適的流程配置設定有助於獲得更佳的分析結果，而配置的選擇因定序平台、資料類型、文庫製備以及速度與準確度之間的平衡等因素而異。以下提供幾種不同配置的範例。

請將 *\<identifier\>* 替換為實際的識別名稱。這些配置有助於自訂分析設定以符合特定的定序參數。在`gconf` 中使用 `-a` 選項可儲存配置設定，使其可在 `gsched` 中透過 `-p` 選項重複調用。有關 `gconf` 指令的更多細節，請參閱第 3.1 節。

  - 針對基於聚合酶連鎖反應法(PCR-based)的WGS數據

<!-- end list -->

```
gconf touch <identifier> --sequencing-protocol pcr -a
```

  - 針對全外顯子定序(WES)數據

<!-- end list -->

```
gconf touch <identifier> --sequencing-datatype wes \
--sequencing-protocol pcr --interval-of-interest \
/opt/geneasic/share/intervals/grch38/<target-region>.bed -a
```

  - 針對NovaSeq或其他使用雙通道 SBS 技術的平台

<!-- end list -->

```
gconf touch <identifier> --sequencing-platform novaseq \
--polyg-trimming -a
```

  - 針對高準確度需求

<!-- end list -->

```
gconf touch <identifier> --operation-mode advanced -a
```

  - 針對從預設流程 default-germline-wes 修改的配置

<!-- end list -->

```
gconf touch <identifier> -s default-germline-wes \
--interval-of-interest /path/to/target-region.bed
```

### 2.2 產生數據集列表檔案

除了使用 EXCEL 等傳統 CSV 編輯器外，`gsched` 指令提供了一種高效的方式來生成分析所需的 CSV 數據，特別適合以符合流程規範的格式管理大量樣本匯入。

  * `-s` 選項：指定 FASTQ 作為輸入格式
  * `-i` 選項：設定瀏覽的起始目錄，在此範例中為 `/home/egs`
  * `-j` 選項：根據指定的正規表示式（如 `(\w+)_R[12]`）從觀察到的數據路徑中自動提取第一個匹配內容，並自動指派為每個樣本的作業名稱。
  * `--as-csv` 標記：將輸出格式化為 CSV

<!-- end list -->

```
gsched -s fastq -i /home/egs -j ‘(\w+)_R[12]’ --as-csv | tee data.csv
```

執行後將出現一個文字介面供檔案選擇。請使用方向鍵與空白鍵選取樣本。並利用 `tee` 指令將輸出導向至 `data.csv`。關於生成樣本列表檔案的更多資訊，請參閱第 3.2 節。

![11](https://hackmd.io/_uploads/HJdUKECEWl.png)

圖 6. 檔案導覽與樣本選取範例

## 3\. 命令行操作詳情

本節詳細介紹各個指令及其可用選項。

### 3.1 *gconf*: 流程配置設定

`gconf` 讓使用者能夠管理數據分析流程配置參數。執行此指令而不帶任何參數時，將顯示所有可用的流程配置。此指令提供多個子指令以利流程管理與自訂流程配置。

  - touch
    此子指令用於透過流程識別碼 (ID(s)) 建立或更新配置參數，讓使用者能靈活地針對不同分析階段配置各種選項。配合使用 `-s` 或 `--schema` 選項，使用者可基於預定義的流程識別碼初始化新的配置。啟用 `-a` 或 `--apply` 選項時，所有更改將套用並儲存至使用者的工作空間。使用者自定義的設定儲存於`/home/$USER/.ngsaap/params.yaml`。

  - inspect
    此子指令用於提供使用者特定流程配置的完整資訊。讓使用者能夠透過流程 ID 檢視詳細的流程配置設定，以進一步了解分析參數。

  - purge
    此子指令提供根據對應流程配置的 ID 移除配置參數。以方便使用者清理不再需要的配置。

系統預定義的所有流程配置是不可變更的，也**無法** 使用 `touch` 或 `purge` 子指令進行修改或刪除。

  - default-germline-wgs-pcrfree
    適用於 PCR-free 文庫製備之 WGS 數據的一般遺傳性基因變異分析設定。

  - default-germline-wgs-pcr
    適用於基於 PCR 文庫製備之 WGS 數據的一般遺傳性基因變異分析設定。

  - default-germline-wes
    適用於採用Capture-based enrichment之 WES 數據的一般遺傳性基因變異分析設定，且啟用了 PCR 偏差校正。

  - novaseq-default-germline-wgs-pcrfree
    專為使用雙通道 SBS 技術（如 Illumina NovaSeq 系列）定序之 PCR-free WGS 數據設計的遺傳性基因變異分析設定，且啟用了序列比對階段的 Poly-G 修剪(trimming)。

  - novaseq-defualt-germline-wgs-pcr
    專為使用雙通道 SBS 技術定序之基於 PCR 的 WGS 數據設計的遺傳性基因變異分析設定，且啟用了序列比對階段的 Poly-G 修剪。

  - novaseq-defualt-germline-wes
    專為使用雙通道 SBS 技術定序(如Illumina NovaSeq Series)，並採用Capture-based enrichment之 WES 數據設計的遺傳性基因變異分析設設定，且啟用了 PCR 偏差校正。

表 1 GeneASIC NGSAAP 全域分析流程配置可用選項

| 選項 | 描述 |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| \--sequencing-platform *\<CHOICE\>* | 設定定序平台。*\<CHOICE\>* 可選項目為 *mgiseq、novaseq 或 element*。預設值為 *mgiseq*。 |
| \--sequencing-datatype *\<CHOICE\>* | 設定定序數據類型。*\<CHOICE\>* 可選項目為 *wgs* 或 *wes*。預設為 *wgs*。|
| \--sequencing-protocol *\<CHOICE\>* | 設定定序序列的文庫製備協定。*\<CHOICE\>* 可選項目為 *pcrfree* 或 *pcr*。預設值為 *pcr*。 |
| \--operation-mode *\<CHOICE\>* | 設定分析模式。*basic* 基礎模式提供最高運算效能，而*advanced* 進階模式提供較高準確度。*\<CHOICE\>* 可選項目為 *basic* 或 *advanced*。預設值為 *basic*。     |
| \--model-path *PATH* | 設定外部機器學習過濾模型的路徑，此設定將覆蓋所有流程配置。預設值為 *null*。     |

表 2 GeneASIC NGSAAP 序列比對器 (Read Mapper) 可用選項

| 選項 | 描述 |
| -------- | -------- |
| \--qual-trimming     | 修剪基因序列末端質量較低的鹼基。套用 --no-qual-trimming 可禁用此功能。     |
| \--polyg-trimming     | 從序列中修剪 poly-G 尾端。套用 --no-polyg-trimming 可禁用此功能。     |
| \--adaptor-trimming *\<CHOICE\>* | 修剪接頭(Adaptor)序列。*\<CHOICE\>* 適用於不同序列修剪方法：*hardclip*、*softclip* 和 *none*。預設值為 *hardclip*。     |

表 3 GeneASIC NGSAAP 變異偵測器 (Variant Caller) 可用選項

| 選項 | 描述 |
| -------- | -------- |
| \--adaptive-pruning |根據觀察到的區域序列讀取數目自動增加單倍型組裝(Haplotype Assemblies)的最低支持度。啟用此參數可降低高覆蓋區域對定序錯誤的靈敏度，從而提高基因變異偵測速度。|
|\--duplicate-removal|移除 PCR 重複序列 (duplication artifacts)。不建議在 PCR (PCR-free) 樣本中使用此項目。|
|\--emit-ref-confidence | 輸出參考基因組信賴度評分 (reference confidence scores)。|
| \--max-reads-per-start *\<INT\>* |此選項控制每個序列比對起始位置保留的最大讀段數。超過此門檻的讀段將被降採樣(Downsampled)。此值應為正整數。預設值為 *4*。|
| \--interval-of-interest *PATH* |參考 *BED* 格式的感興趣區間，指定用於分析的基因組區域。*\<PATH\>* 應設定為該 *BED* 檔案的絕對路徑，以確保與參考基因組組裝對齊。預設值為 *null*。 |
| \--database-annotation |指定變異資料庫的路徑，用於報告指標。預設值為 *[]* |

表 4 結構變異 (SV) 偵測器可用選項

| 選項 | 描述 |
| -------- | -------- |
| \--manta-min-candidate-variant-size *\<INT\>* |用於指定在候選變異偵測與最終輸出結果中被考慮的結構變異（SV）與插入/刪除（indel）的最小片段大小。預設值為*8*。|
| \--manta-min-edge-observations *\<INT\>* |過濾圖形邊緣(Graph Edges)，移除支援觀測值少於 *\<INT\>* 的邊緣。預設為 *3*。|
|\--manta-graph-node-max-edge-count *\<INT\>*|當兩個節點的邊緣計數皆高於指定值時中限制節點間評估的邊緣數量。預設為 *10*。|
| \--manta-min-candidate-spanning-count *\<INT\>*| 此選項設定侯選發現報告中，SVs 和 indels 所需的最小跨越支援讀數。預設為 *3*。|
|\--manta-min-scored-variant-size *\<INT\>* | 進行候選結構變異識別後，應進行評分並報告的 SVs 和 indels 之最小設定值（以鹼基對為單位）。預設值為 *50*。|
|\--manta-min-diploid-variant-score *\<INT\>* | 設定變異被包含在二倍體 VCF 中的最小 QUAL 分數。預設值為 *10*。 |
| \--manta-min-pass-diploid-variant-score *\<INT\>* | 設定二倍體 VCF 中將變異標記為已過濾（Filtered）的 QUAL 評分門檻。預設為 *20*。 |
|\--manta-min-pass-diploid-GT-score *\<INT\>* | 設定在二倍體 VCF中包含單一樣本的最低基因型質量(Genotype Quality)評分。預設為 *15*。 |
|\--manta-min-somatic-score *\<INT\>* | 設定體細胞變異包含在體細胞 VCF 中的最低質量評分。預設為 *10*。 |
| \--manta-min-pass-somatic-score *\<INT\>* | 設定在體細胞 VCF 中過濾體細胞變異的品質評分門檻。預設為 *30*。 |
| \--manta-enable-remote-read-retrieval-for-insertions-in-germline-calling-modes *\<INT\>* |利用低比對質量的遠端位置檢索配對讀段，以改善遺傳性插入變異的組裝。*\<INT\>* 1為啟用設定，0 為禁用設定。預設值為 *1*。 |
|\--manta-enable-remote-read-retrieval-for-insertions-incancer-calling-modes *\<INT\>* |利用低比對質量的遠端位置檢索配對讀段，以改善體細胞變異偵測模式中插入變異的組裝。*\<INT\>* 1為啟用設定，0為禁用設定。預設為 *0*。|
| \--manta-use-overlap-pair-evidence *\<INT\>* | 定義是否將重疊的配對讀段 (Read Pairs) 作為證據。*\<INT\>* 1為包含設定，0 為排除設定。預設值為 *0*。 |
| \--manta-enable-evidence-signal-filter *\<INT\>* | 對證據信號不明顯的候選變異啟用過濾器。*\<INT\>*1為啟用設定，0 為停用設定。預設值為 *1*。 |

表 5 短片段重複序列 (Short tandem repeat, STR) 大小估計可用選項
| 選項 | 描述 |
| ------ | ----------- |
| \--repeat-region-extensionlength *\<INT\>* |設定搜尋目標區域周圍讀段的片段長度。預設值為 *1000*。 |
| \--repeat-variant-catalog *PATH* |設定用於變異基因型鑑定（Genotyping）的 JSON 目錄路徑。。預設為 */opt/geneasic/database/ehunter/variant\_catalog.{hg}.json* |

表 6 HLA 分型可用選項
| 選項 | 描述 |
| ------ | ----------- |
|  \--hla-sample-population *\<CHOICE\>* | 此選項設定 HLA 分型的族群。*\<CHOICE\>* 可選 *Asian (亞洲人), Black (黑人), Caucasian (白種人), Unknown (未知)，以及nonuse不使用*。預設值為 *Unknown*。 |
| \--hla-exon-typing | 啟用 HLA 的外顯子分型。 |
| \--hla-min-maf *\<FLOAT\>*| 設定 HLA 分析的最小等位基因頻率(minimum minor allele frequency)。預設值為*0.1*。|

表 7 ROH 可用選項
| 選項 | 描述 |
| ------ | ----------- |
|\--roh-min-size *\<INT\>* | 設定報告中顯示最小變異尺寸（單位:百萬鹼基對, Mb）。預設值為 *300*。 |

表 8 結構變異註釋可用選項
| 選項 | 描述 |
| ------ | ----------- |
| \--annotsv-sv-min-size *\<INT\>*| 設定用於註解結構變異尺寸 (bp)。預設值為 *50*。 |

表 9 變異判讀報告可用選項
| 選項| 描述 |
| ------------------- | ----------- |
|\--transcript-annotation-mode *\<STR\> [STR …]* |指定轉錄本註釋資料庫。*\<STR\>* 可選項目為 Ensembl 與 Refseq。預設為 *[ Ensembl ]*。 |
| --disease-trait *\<CHOICE\>* | 指定用於註釋的疾病特徵。*\<CHOICE\>* 在遺傳性變異分析中可選 *Universal (通用)* 與 *Stroke (中風)*，在體細胞變異分析中可選 *Breast (乳癌), Lung (肺癌), Pancreatic (胰臟癌), Colorectal (大腸直腸癌), Chronic\_Myelogenous\_Leukemia (慢性\骨髓性\白血病), Cholangiocarcinoma (膽管癌)*。預設值為 *Universal*。 |
|\--annotate-all-disease-related-genes| 註釋所有與疾病相關的基因。啟用此設定後，將產生名為 *\<jbname\>*\_all\_annotation.csv 的 *.xlsx* 檔案。*預設* 僅註釋致病性(Pathogenic)變異。 |
|\--disease-related-gene-list *PATH* |指定自定義基因列表的路徑，使分析聚焦於特定的疾病相關基因。當預設值為 *null* 時，則參考 */opt/geneasic/tertiary/\<assembly\>/GeneList/omimGene\_list.txt*。預設值為 *null*。 |
|\--secondary-finding-gene-list *PATH* |指定次要基因列表的路徑。當預設值為 null時，則參考 */opt/geneasic/tertiary/\<assembly\>/GeneList/secondaryFinding.txt*。預設值為 *null*。 |
|\--carrier-screening-gene-list *PATH*| 指定帶因者篩檢(Carrier Screening)基因列表的路徑。若預設值為 null 時，則參考 */opt/geneasic/tertiary/\<assembly\>/GeneList/carrierScreening.txt*。預設為 *null*。 |
| \--clonal-gene-list *PATH* | 指定單株性造血基因列表的路徑。當預設值為 null 時，則參考 */opt/geneasic/tertiary/\<assembly\>/GeneList/clonalGene\_list.txt*。預設為 *null*。 |
| \--inhouse-snv-db *PATH* |指定用於 SNV 過濾的內部整合資料庫路徑。預設為 *null*。 |
|\--inhouse-snv-cutoff *\<FLOAT\>* |設定 SNV 過濾門檻值（需於 0 與 1 之間）。預設為 *0.2*。 |
|\--inhouse-sv-db *PATH* | 指定用於 SV 過濾的內部整合資料庫路徑。預設為 *null*。 |
| \--inhouse-sv-cutoff *\<FLOAT\>* |設定 SV 過濾門檻值（需介於 0 與 1 之間）。預設為 *0.2*。  |

### 3.2 gsched: 樣本資料配置

`gsched` 指令旨在協助使用者建立用於基因組數據分析的 YAML 檔案。它能讓使用者配置分析參數、選取匯入數據、指定輸出項，並生成 .yaml 格式的任務清單。

使用 CSV 檔案產生任務列表 `batch.yaml` 的範例：

```
#generate a task list with a CSV file
gsched -b batch.yaml \
-p default-germline-wgs-pcrfree -s fastq -t bam vcf \
-i /home/egs - f data.csv -o results -a
```

  - **配置分析**
    允許使用 `-g` 選擇參考基因組版本，並使用 `-p` 指派分析流程配置

  - **匯入資料**
    指定用於分析的匯入數據目錄和/或格式

  - **指派輸出**
    指定所需的分析輸出項目並指派輸出目的地

  - **產生任務列表**
    建立包含所有配置內容的 YAML 檔案
    
表 10.1 分析配置可用選項

| 選項 | 描述 |
| ------ | ----------- |
| -g, \--assembly *\<VERSION\>* |  控基因組組裝所使用的參考基因組。*\<VERSION\>* 應指定為 hs38d2，hs37d5，hg19或hg38。預設值為 *hs38d2*。|
| -p, \--pipeline *\<ID\>* | 指定分析所使用的流程配置。輸入 *-p 或 --pipeline* 後連按兩次 TAB 鍵可查看可用選項。也可使用*gconf insepct* 指令查看。|

下表說明了在分析配置中，透過 `-g, --assembly`  參數可選擇的基因組版本及其技術特性：

表 10.2 參考基因組分析配置選項

| 選項 | 描述 |參考文獻 |
| ------ | ----------- |----------- |
| hs38DH (預設) |基於 hs38DH 的優化版本。除了包含完整的 GRCh38 分析集（含誘餌序列與 HLA）外，更採用了調整後的屏蔽 (Masking) 方法處理替代位點 (ALT contigs)，並對特定策略區域(Strategic regions)進行硬屏蔽，是全基因組定序 (WGS) 的最佳推薦選擇。  |Li H. (2013). arXiv:1303.3997; Illumina DRAGEN Guide|
| hs37d5 |源自 GRCh37 的完整分析集。此版本額外包含人類誘餌序列 (Decoy sequences) 與 EBV 病毒序列，能有效吸收非目標片段，從而減少變異偵測的假陽性。為臨床診斷的標準版本。 |1000 Genomes Consortium. (2015). *Nature*|
| hg19 |UCSC 標準 GRCh37 參考序列。作為早期標準版本，本組裝亦包含誘餌序列以改善序列比對品質，適用於需與歷史數據對照的分析。 |Schneider et al. (2017). *Genome Research*|
| hg38 |UCSC 標準 GRCh38 參考序列。包含主染色體與替代位點 (ALT)，但不包含 hs38d2 隨附的人類誘餌序列 (Decoys)，適用於一般視覺化或不需要誘餌序列輔助的特定分析。 |Schneider et al. (2017). *Genome Research*|

不同來源的參考基因組在命名規則與組成上有顯著差異。使用者可透過下表快速識別並驗證所使用的參考檔案：

表 10.3 支援之人類參考基因組總表

| 資料庫組裝版本 | 來源 | contig 數量 |前綴風格(Prefix Style) |誘餌序列標籤 (Decoy Tag) |
| ------| -------- | -------- |-------- |-------- |
| hs38DH | NCBI     |3366    | chr1, chrUn     | chrUn*_decoy     |
| hs37d5 | NCBI/1000 Genome | 86     | 1, GL*     | hs37d5     |
| hg38   | UCSC     | 455     | chr1, chrUn     | N/A     |
| hg39   | UCSC     | 93     | chr1, chrUn     | N/A     |

*N/A:不適用


為了提升比對精度，高品質的分析集（如 hs38DH）由多種功能的序列組成。以下說明各類元件的定義與目的：

表 10.4 參考基因組元件解析 (以 hs38DH 為例)

| 類別 | contig數量 |描述  |
| --- | ---------  |---  |
| 主參考序列 (Main Reference) | 25  |包含染色體 chr1-22、X、Y 以及線粒體 (chrM)，是人類基因組的核心區域。 |
| 未定位序列 (Unlocalised Sequences) | 42  |標示為 chr*_random。已知屬於特定染色體，但具體座標尚未確定的序列。|
| 未放置序列 (Unplaced Sequences) | 127  |標示為 chrUn*。確信屬於人類基因組，但無法確定其源自哪條染色體的序列。|
| 替代位點 (Alternative Loci) | 261  |標示為 chr*_alt。代表人類族群中高度多態性的區域（如免疫系統相關基因），用於「替代位點感知 (Alt-aware)」比對，防止假陽性。|
| 愛潑斯坦-巴爾病毒序列 (EBV sequence) | 1  |標示為 chrEBV。用於吸收樣本中潛在的病毒 DNA 片段，避免其干擾人類基因比對。 |
| 誘餌序列 (Decoy Sequences) | 2385  |標示為 chrUn*_decoy。作為「匯漏 (Sink)」，吸引容易引起誤比對的重複序列或非參考片段，是提升 WGS 準確度的關鍵。|
| HLA 序列 (HLA Sequences) | 525  |標示為 HLA-*。專門針對高度複雜的 HLA 區域設計的等位基因序列，對於精準的 HLA 分型至關重要。 |


表 11 數據匯入可用選項
| 選項 | 描述 |
| ------ | ----------- |
| -s, \--source *\<FORMAT\>* | 指定輸入來源的數據格式。\<FORMAT\> 可選單項或多項，說明如下。<br>• fastq<br> 設定 FASTQ 為雙端定序數據的輸入格式。檔案應分開儲存，讀段片段 1 擴展名如 1.fastq.gz、1.fq.gz 或 1\_001.fastq.gz 的副檔名，讀段片段 2 則如 2.fastq.gz、2.fq.gz 或 2\_001.fastq.gz。<br>• bam, vcf<br>指定已準備好分析的 BAM 和 VCF 檔案作為輸入格式。檔案應分別以 .bam 和 .vcf 結尾，可包含索引檔案 (.bai 或 .tbi)。<br>• sv<br>指定已準備好分析的 SV 檔案共以.sv.vcf.gz 結尾的檔案作為輸入格式，可包含.tbi 索引檔案。<br>• px<br>從 .px.json 檔案載入樣本資訊。結構包含以下欄位，預設值為空白。<br>• Order Number: 訂單的唯一識別碼。<br>• Client Order Number: 由客戶分配的追蹤識別碼。<br>• Ordering Physician: 開立檢測的醫師姓名。<br>• Physician ID: 醫師的唯一識別碼。<br>• Patient Name: 患者姓名。<br>• Patient ID: 患者的身份識別碼。<br>• Birthday: 患者出生日期。<br>• Sex: 患者性別。<br>• Age: 樣本採集時的患者年齡。<br>• Sample Type: 採集的檢體類別（如血液、組織）。<br>• Sample Source: 樣本來源（如特定解剖部位）。<br>• Collect Date: 樣本採集日期。<br>• Sequencing Type: 執行的定序類型（如 WGS、WES）。<br>• Symptom: 與患者相關的症狀或狀況。<br>• Note: 與樣本相關的額外備註或說明。 |
| -i, \--import-dir *\<DIR\>* | 在指定目錄中啟用檔案導覽以選取分析數據。使用方向鍵移動、空白鍵選取、ENTER 鍵結束選取並匯出。 |
|-j, \--jbname *\<REGEX\>*|根據符合檔案絕對路徑部分的特定正規表示式（Regular Expression）指派分組標籤。預設使用 ([^\/]+)\/[^\/]+$，將父目錄名稱指派為作業名稱（Job Name），適用於每個樣本存放於獨立資料夾的架構。|
| -x, \--exclude *\<DIR\>*|在 -s 選項啟用的檔案導覽中排除無關的檔案或目錄，從而精簡候選選擇列表。 |
| \--as-csv | 與 -i、-s（及 -j）配合使用以生成 CSV 樣本列表。可配合 tee 指令將輸出導向至 .csv 檔案。  |
| \--as-tsv | 與 -i、-s（及 -j）配合使用以生成 TSV 樣本列表。可配合 tee 指令將輸出導向至 .tsv 檔案。|
| -f, \--files-from *\<DIR\>* |此選項直接從 CSV 或 TSV 樣本列表匯入資料。樣本列表必須遵循以下欄位： <br>\<jbname\>,\<source-type\>/\<index\>,\<source-path\>,\<suffix-size\><br>• jbname: 特定輸入數據及後續分析的作業名稱<br>• source-type: -s 選項中允許的項目<br>• index: 特定指派對應來源的類型<br>• source-path: 輸入檔案的位置<br>• suffix-size: gsched 產生的可選欄位，用於解決歧義<br><br>可透過 *--as-csv* 或 *--as-tsv* 選項，連同 *-i* 和 *-s* 選項，以及可選的 *-j* 選項產生CSV 或 TSV 樣本列表。|

為了準確提取群組標籤(Group Label)，使用者可以使用 regex101.com 等線上工具測試其正規表示式。

如圖 7 所示，運算式 `(\w+)` 成功從各個樣本的測試字串中提取了sample1、sample2 和 sample3（綠色highlight字體顯示）。若要在最終確定前預覽分組結果，請在執行 `gsched` 指令時啟用 `--as-csv` 或 `--as-tsv` 指令。此預覽步驟有助於在繼續執行前，確保作業名稱（Job Name）指派的準確性與清晰度。
![13](https://hackmd.io/_uploads/r123MNGrWx.png)

圖 7. 使用線上正規表示式測試工具測試分組標籤之範例

表 12 結果輸出可用選項
|  選項  | 描述         |
| ----------- | ---------------------------------- |
| -t, \--target *\<CHOICE\>* |  指定所需的分析與輸出結果。*\<CHOICE\>* 可包含一個或多個選擇。<br>•fastq<br>建立指向來源 FASTQ 檔案的軟連結(symbolic link)。<br>• bam<br>使用 GeneASIC NGSAAP 比對器執行短讀長序列比對，產出 BAM 檔案及.bai 索引檔案。若配合 -s 選項且偵測到現有 BAM 檔，則改為建立軟連結。<br>• vcf<br>使用 GeneASIC NGSAAP 變異偵測器執行類似 GATK 短變異偵測，產出 .vcf.gz 檔案及 .vcf.gz.tbi 索引檔。若配合 -s 選項且偵測到現有 VCF 檔，則改為建立軟連結。<br>• sv<br>調用 Manta 偵測結構變異，產出 .sv.vcf.gz 及 .sv.vcf.gz.tbi 檔案。如果與 -s 選項一起指定時偵測到現有的 .sv.vcf.gz 檔案，此選項將建立指向輸入檔案來源的軟連結。<br>• px<br>建立指向來源輸入 .px.json 檔案的軟連結。<br>• cnv<br>使用 ASCAT R 套件執行拷貝數變異分析，並由 SLMSuite 進行分段（Segmentation）。啟用此選項會輸出 *.segment.SLMed.txt* 檔案。<br>• upd<br>執行潛在單親二倍體（Uniparental Disomy）提取，產出 .upd.txt 候選清單供變異判讀使用。<br>• smn<br>執行針對 SMN1、SMN2 及 SMN2Δ7-8 的拷貝數變異分析。產出 *.smn.txt* 檔案。<br>• roh<br>使用 GeneASIC ROH 偵測器執行純合子區段分析，產出 *.roh.bed* 檔案。<br>• str<br>使用 ExpansionHunter 進行短片段重複序列大小估計。預設變異目錄位於 */opt/geneasic/config/\<assembly\>\_variant\_catalog.json*，其中 assembly 可以是 hs38d2 或 hs37d5。結果儲存於 *.repeat.vcf* 檔案。<br>• hla<br>使用 SpecHLA 進行人類白血球抗原分型。摘要匯出至 *.hla.txt*。<br>• mito<br>使用 GATK Mutect2 在粒線體模式下進行粒線體變異偵測，接著使用 GATK FilterMutectCalls。輸出檔案包括 *.mitochondria.filtered.vcf.gz* 檔案及其索引檔。<br>• ch<br>使用 GATK Mutect2 以腫瘤僅有 (tumor-only) 模式進行單株系造血分析。流程包括 GATK GetPileupSummaries、CalculateContamination 和 FilterMutectCalls，接著進行遺傳性變異的嚴李格過濾。結果儲存於 *.ch.filtered.clean.vcf.gz*，連同其索引檔和 *.stats* 檔案。<br>• holmes<br>使用 GeneASIC Holmes 進行自動化臨床遺傳性變異註釋與分類，產出為 *.tsv.gz* 檔案。<br>• holmes\_vcf<br> 將 Holmes 的註釋與判讀資訊附加至 VCF 檔，產出 *.holmes.vcf.gz*。此格式與 JBrowse 2 相容，可用於渲染 Ensembl VEP 使用的 CSQ 欄位。<br>• annotsv<br>執行結構變異註釋與排序，產出 *.sv.annotated.tsv* 檔案。<br>• report<br>使用 ANNOVAR 以及 gnomAD、ClinVar、CADD、Revel 和 GWAS 資料庫，針對疾病相關基因、ACMG 建議的次要發現和帶因者篩檢基因區域中的變異偵測進行變異判讀。它還包括利用 PharmCAT 進行的額外藥物基因組學臨床註釋，以及透過PGS catalog 進行的多基因風險評分。結果彙整成 PDF 和 XLSX 檔案，如果在 gconf 指令中指定了 *--annotate-all-disease-related-genes* 指令，則額外產出 \_all\_annotations.csv檔案。<br>• report\_csv<br>匯出變異判讀過程中產生的所有 CSV 檔案。
| -T, \--default-targets *\<CHOICE\>* | 為簡化 *-t* 選項指定分析和輸出項目的操作，*-T* 允許使用者透過單一輸入選取多個選項。*\<CHOICE\>* 可選項目：<br>• germline-basic<br>指定 bam, vcf, holmes 和 report 為目標輸出檔案<br>• germline-full<br>指定 *bam, vcf, cnv, upd, smn, str, sv, hla, roh, mito, ch, holmes, annotsv,report* 為目標輸出檔案 |
| -o, \--export-dir \<DIR\>  |   指定輸出目錄。若未指定，預設為當前目錄。 |
|  \--group-by-jbname  |    根據作業名稱（Job Name）將結果整理至各別資料夾。  |
|  \--flatten-directory-structure  | 攤平巢狀目錄結構，將所有輸出檔案儲存在單一目錄中。  |

表 13 任務列表產生可用選項
| 選項 | 描述 |
| ------ | ----------- |
| -b *PATH* | 指定 YAML 格式的批次配置檔案路徑。如果未提供路徑，預設存為當前目錄下的 batch.yaml。該檔案可多次覆寫以在單一檔案中配置不同設定的樣本。 |
| \--jbname-suffix | 在衝突的作業名稱後添加後綴字串，以解決命名重複的歧義。 |
|-a, \--apply | 離開預覽模式並儲存配置。 |
| \--overwrite | 替換現有的作業名稱。 |
| \--skip |  跳過名稱衝突的作業。 |

### 3.3 gsub: 作業提交

`gsub` 指令允許使用者將透過 `gsched` 建立的 `batch.yaml` 中指定的任務提交至 GeneASIC NGSAAP 作業佇列。要啟動樣本分析任務的提交，請執行以下指令：

```
gsub batch.yaml
```

執行後，此指令將為您的提交提供一個唯一的批次 ID，建立 `batch.yaml` 檔案中定義的相應檔案和目錄，並在當前工作目錄中產生記錄檔 (log files)。要檢視作業狀態並存取詳細的執行資訊，請參閱第 3.4 節關於使用 *gwatch* 指令的指南。

表 14 作業提交可用選項
| 選項 | 描述 |
| ------ | ----------- |
| -a, \--array *\<STR\>* | 允許使用者利用從 1 開始的陣列索引（Array Index）選取特定樣本進行提交。選項參數可以是逗號分隔的值、連續的索引值範圍，以及可選的步長，如下例所示：<br>• -a 1-10<br>提交索引值在 1 到 10 之間的作業陣列<br>• -a 1,3,5,7<br>提交索引值為 1, 3, 5 和 7 的作業陣列<br>• -a 1-7:2 提交索引值在 1 到 7 之間且間隔為 2 的作業陣列。預設設定中於 `batch.yaml` 定義的所有任務將提交至 GeneASIC NGSAAP 作業佇列。 |
| -w, \--nodelist *\<STR\>* | 允許使用者以逗號分隔字串格式指定特定的運算主機列表。分析作業將根據需要排程在這些指定的主機上，以滿足資源需求。列表中的任何重複節點名稱將被忽略，確保每個節點僅計算一次。 |
|-x, \--exclude *\<STR\>* | 指定需排除的主機列表，確保特定節點不被用於此次分析。 |

### 3.4 gwatch: 作業佇列檢視器

`gwatch` 指令讓使用者能輕鬆監控作業狀態並存取詳細的運行資訊。直接執行 `gwatch` 指令而不帶任何參數，即可啟動文字介面的作業狀態檢視器。

在檢視器中，紀錄分為「進行中（In Progress）」與「已完成（Finished）」。使用者可選擇以精簡 (Concise) 或詳細 (Verbose) 模式檢視紀錄，並使用方向鍵與空白鍵方便地在兩者間切換。

在精簡模式下，每條紀錄皆由「BATCHID_TASKID」格式的作業 ID 唯一識別。此外，每個任務皆關聯一個由 `--gsched` 指令解析的作業名稱，並標註以下狀態之一以表示進度：

  - RUNNING (執行中)：表示任務正在積極執行的任務。
  - PENDING (等待中)：表示任務在佇列中等待可用資源開始的任務。
  - COMPLETE (完成)：表示任務已成功完成執行。
  - FAILED (失敗)：標記在執行過程中遇到錯誤或問題且未按預期完成的任務。
  - CANCELLED (取消)：識別在完成前被人為終止的任務。

此外，精簡模式下的每條紀錄都會顯示作業的已耗用時間，讓使用者快速掌握任務執行的時長。

在詳細模式下，使用者不僅可以查看作業所有權、運算節點名稱、提交時間、開始與結束時間點等資訊，還能獲取分析流程中各個階段（Stage-level）的運行資訊。

預設情況下，`gwatch` 會檢索 4 小時內的作業執行記錄。使用者可以使用 `--within` 選項指定所需的檢索範圍，例如 `--within 8` 以查閱過去 8 小時內的作業執行歷史記錄。

### 3.5 gcancel : 作業取消

`gcancel` 指令用於終止受 GeneASIC NGSAAP 控制的作業。

表 15 作業終止可用選項
| 選項 | 描述 |
| ------ | ----------- |
| \--batch *\<ID\> [ID ...]* | 允許使用者透過指定一個或多個批次 ID 來終止該批次的所有任務。 |
| \--jobid *\<ID\> [ID ...]* |允許使用者透過指定一個或多個作業 ID 來終止特定任務。 |
|\--user *\<STR\>* | 將取消操作限制為特定使用者擁有的作業。 |
| \--me | 將取消操作限制為當前使用者擁有的作業。 |

### 3.6 gviz : JBrowse 引導程式

`gviz` 指令是專為 JBrowse 網頁應用程式設計的引導程式（Wizard），JBrowse 是一款用於視覺化基因組定序數據的工具。使用者可以輕鬆匯入分析結果，包括序列比對 檔案(BAM)、基因變異偵測檔案 (VCF)，以及各種序列判讀格式（如 BED 或 GFF3檔案）的特徵軌跡 (feature tracks) 與註釋。若要選取用於視覺化的基因組軌跡，請執行下列指令啟動選取介面：

```
gviz -s /nas/homes/$USER/<results> -t /path/to/jbrowse
```
在此指令中，使用 -s 選項指定包含分析結果的目錄，並使用 -t 選項指定 JBrowse 目錄。請將 <results> 替換為分析結果的實際路徑。

![16](https://hackmd.io/_uploads/ryS1QQXr-l.png)

圖 8. 軌跡選取器 (Track Selector) 範例

當 /path/to/jbrowse 資料夾建立後，您可以使用下列指令啟動本地網頁伺服器：
```
npx serve -S -l <port> /path/to/jbrowse
```

該指令會在指定的連接埠（Port）上建立進入點。按下 Ctrl 鍵並點擊生成的連結，即可開啟 JBrowse 並探索您的分析結果。

![18](https://hackmd.io/_uploads/BkEam77rbe.png)

圖 9. JBrowse 進入點範例

若要匯入額外的軌跡進行瀏覽，放置於 jbrowse/<results> 資料夾中的任何內容皆可透過相對於 jbrowse 目錄的路徑進行存取。

如需更完整的操作資訊，請參考 JBrowse 官方文件：https://jbrowse.org/jb2/docs/user_guide/。

![19](https://hackmd.io/_uploads/rk18NmXSWx.png)

圖 10. 匯入 BED 檔案的步驟





### 3.7 gversion : 程式版本檢視器

`gversion` 指令顯示 GeneASIC NGSAAP 的全面版本資訊，包括硬體、韌體、命令列工具、第三方函式庫和基因組資料庫版本。

![20](https://hackmd.io/_uploads/Sk8hV77rWg.png)


圖 11. GeneASIC NGSAAP 版本資訊範例

### 3.8 acctmgr : 使用者帳號管理員

GeneASIC 帳號管理員 (acctmgr) 提供了一個用於管理 GeneASIC NGSAAP 環境內使用者帳號與群組的介面。請確認您擁有執行這些指令的權限，並在刪除使用者或群組時保持謹慎，因為此動作是不可逆的。

  - acctmgr list
    此指令列出系統內所有現有的使用者帳號與群組，提供當前使用者管理配置的總覽。
  - acctmgr +g \<groupname\> [\<gid\>]
    此指令允許管理員建立名為 <groupname> 的新使用者群組，並可選擇性指定群組 ID (<gid>)。若未指定，系統將自動指派。
  - acctmgr +u \<username\> \<groupname\> [\<uid\>]
    此指令允許管理員將指定的 <username> 使用者加入現有群組，並可選擇性指定使用者 ID (<uid>)。執行後，系統將提示管理員為該使用者設定登入密碼。
  - acctmgr +G \<username\> \<groupname\>
    將現有使用者新增至次要群組。這通常用於授予額外權限。
    **範例：** `acctmgr +G john fpgauser` 將使用者 john 新增至 fpgauser 群組，授予他們使用 FPGA 分析基因組資料的權限。
  - acctmgr -u \<username\>
    此指令從系統中移除指定的 <username> 使用者帳號。
  - acctmgr -g \<groupname\>
    此指令從系統中刪除指定的 <groupname> 使用者群組。
  - acctmgr -G \<username\> \<groupname\>
    從指定的次要群組中移除使用者，而不刪除使用者的帳號。

## 4\. 修訂歷史

| 版本 | 日期 | 變更 |
| ------ | --- | ----------- |
| 1.1.0\_b | 2026 Jan | 大幅更新, 修改文字及更新圖片link |
| 1.1.0\_b | 2025 Dec | 小幅更新 |
| 1.1.0\_a| 2024 Nov |  更新 GeneASIC NGSAAP All-in-One 版本的完整功能  |
| 1.0.0  | 2023 Mar|  初始版本   |