---
title: GeneASIC NGSAAP All-in-One 命令列工具使用者手冊

---



```
聯絡資訊:
GeneASIC Technologies Corp.
info@geneasic.com
```

版本 1. 1 .0\_b

# GeneASIC NGSAAP All-in-One 命令列工具使用者手冊

## 閱讀使用者手冊之前

本使用者手冊適用於 GeneASIC NGSAAP All-in-One v1. 1 .0 版本。在使用 GeneASIC NGSAAP 之前，請務必詳細閱讀本手冊。

本文件中包含的所有圖片僅作為示意範例。實際軟體內容可能略有不同。
如需準確資訊，請參閱您所取得的軟體版本。

GeneASIC Technologies Corp. 已採取措施確保本手冊的準確性。儘管如此，GeneASIC Technologies Corp. 不對手冊中的任何錯誤或遺漏承擔責任，並保留更新手冊和軟體的權利，以提升其可靠性、效能或設計。

本手冊中提及的商標和品牌名稱均為其各自所有者的財產。

如有任何進一步問題，請聯繫 GeneASIC Technologies Corp.。

© 2025 GeneASIC Technologies Corp. 保留所有權利。

## 目錄

  - 1.  簡介
    <!-- end list -->
      - 1.1 運算架構
      - 1.2 基因組變異分析流程
  - 2.  WGS 分析範例
    <!-- end list -->
      - 2.1 自訂流程配置設定
      - 2.2 產生資料集列表檔案
  - 3.  命令列操作詳情
    <!-- end list -->
      - 3.1 gconf: 流程配置設定
      - 3.2 gsched: 樣本資料配置
      - 3.3 gsub: 作業提交
      - 3.4 gwatch: 作業佇列檢視器
      - 3.5 gcancel : 作業取消
      - 3.6 gversion : 程式版本檢視器
      - 3.7 acctmgr : 使用者帳號管理員
  - 4.  修訂歷史

## 1\. 簡介

GeneASIC NGSAAP（次世代定序分析加速平台）是一套基於 FPGA（現場可程式化邏輯閘陣列）的硬體加速 NGS 二級分析系統。除了基因組分析能力外，GeneASIC NGSAAP 的設計還考量了可擴展性與易用性，採用串流架構在分散式運算框架中即時處理資料。

### 1.1 運算架構

GeneASIC NGSAAP 採用分散式運算架構，優化了大型基因組資料集的處理。每個 GeneASIC NGSAAP 節點都配備了 FPGA 加速卡，以提升讀段比對（read mapping）和變異位點呼叫（variant calling）等任務的處理速度。這種架構不僅加速了複雜的基因組分析，還釋放了 CPU 資源用於其他任務，例如進階變異發現、資料庫註釋和臨床意義判讀。

![fig1](https://hackmd.io/_uploads/HkOWtuCMZe.png)
圖 1. GeneASIC NGSAAP 運算架構

資源分配和任務管理由 SLURM、Snakemake 和 GeneASIC 流程控制器無縫協調，為任何規模的專案提供可擴展性。SLURM 允許使用者將作業排入佇列，並在節點可用時自動分派。流程控制器與 Snakemake 結合，優化了 FPGA、CPU、記憶體和磁碟資源的使用，即使在消費級機器上也能高效管理多樣化的任務。GeneASIC NGSAAP 亦支援多使用者環境，允許數名使用者同時提交作業，無需手動排程或擔心節點可用性。這些功能顯著提升了 GeneASIC NGSAAP 在大規模專案中的可用性，將基因組資料分析的容量與效率最大化。

GeneASIC NGSAAP 內的資料流允許直接從 USB 隨身碟、MicroSD 卡、網路附加儲存（NAS）或支援網路檔案系統（NFS）的資料中心等來源串流輸入。此功能實現了並行資料傳輸與處理。分析完成後，結果將安全地傳輸至使用者指定的儲存位置。所有操作均在區域網路（LAN）內的本地端進行，確保最大的基因組資料隱私，並最大限度地減少預取大型資料集（如 FASTQ 檔案）的需求。

總而言之，GeneASIC NGSAAP 利用 FPGA 加速核心分析，並利用 SLURM 進行高效、全面、安全且可擴展的基因組資料分析。多使用者支援與簡化的資料流使 GeneASIC NGSAAP 成為加速基因組發現且無運算瓶頸的強大工具。

### 1.2 基因組變異分析流程

由 FPGA 驅動的 GeneASIC NGSAAP 流程包含兩個主要階段用於短變異（short variant）分析：讀段比對與變異位點呼叫。在 All-in-One 版本中，包含了額外的分析與變異判讀，例如結構變異分析、變異註釋等。

**第 1 階段：讀段比對 (Read Mapping)**

此過程始於將原始定序讀段比對至參考基因組。此階段包含關鍵的預處理步驟，包括修剪讀段以消除低品質鹼基或接頭（adaptors），以及根據其位置座標組織讀段。此階段的產出是一組可用於分析的讀段，為後續的變異位點呼叫奠定基礎。

**第 2 階段：變異位點呼叫 (Variant Calling)**

第二階段透過 FPGA 硬體加速處理短變異呼叫。接著應用機器學習技術來改進和過濾識別出的變異。根據文庫製備的設定，可選擇性地應用 PCR 偏差校正。輸出結果包括 VCF 格式的 SNP 和 INDEL 詳細目錄。此外，在 All-in-One 版本中，此階段還執行 CPU 驅動的分析，如結構變異呼叫（Manta）、拷貝數分析（ASCAT、SLM 和 SMNCopyNumberCaller）、潛在單親二倍體提取、短串聯重複序列大小估計（ExpansionHunter）、HLA 分型（SpecHLA）、株系造血分析（GATK Mutect2）以及粒線體變異呼叫（GATK Mutect2 粒線體模式）。

**第 3 階段：變異判讀 (Variant Interpretation)**

在最後階段，變異會被註釋並遵循 ACMP-AMP 變異分類指南進行嚴格分析，而結構變異則使用 AnnotSV 進行判讀。此外，此階段也完成 ACMG 建議的帶因者篩檢和次要發現、藥物基因組學註釋以及純合子區段（ROH）分析。所有結果最終彙整為 XLSX 和 PDF 格式的報告。

![fig2](https://hackmd.io/_uploads/S1YQsORfWg.png)
圖 2. GeneASIC NGSAAP 工作流程

## 2\. WGS 分析範例

以下是分析 WGS 資料的逐步範例。

### 步驟 1. 準備工作區與資料集

```
#建立一個工作目錄
mkdir example
cd example
```

對於批次分析，建議建立一個 CSV 檔案，指定作業名稱與相關的 FASTQ 檔案路徑。一個用於分析的 CSV 範例 `data.csv` 如下：

```
sample1,fastq/0,/home/egs/sample1_R1.fastq.gz
sample1,fastq/1,/home/egs/sample1_R2.fastq.gz
sample2,fastq/0,/home/egs/sample2_R1.fastq.gz
sample2,fastq/1,/home/egs/sample2_R2.fastq.gz
```

`fastq/0` 和 `fastq/1` 分別代表雙端定序資料的R1 和R2。請確保 `data.csv` 儲存於資料集工作區中。

### 步驟 2. 產生任務列表

```
#使用 CSV 檔案產生任務列表
gsched -b batch.yaml\
-p default-germline-wgs-pcrfree -s fastq -t bam vcf\
-i /home/egs -f data.csv -o results -a
```

此指令會產生一個 `batch.yaml` 檔案，將 `data.csv` 中指定的輸入資料與名為 `default-germline-wgs-pcr-free` 的流程配置設定關聯起來。`-s` 選項表示輸入需要 FASTQ 資料，而 `-t` 選項指定輸出為 BAM 和 VCF 檔案。`-i` 選項設定 `data.csv` 的匯入目錄，`-o` 將輸出目錄定義為 `results`。使用 `-a` 選項套用所有設定，指令即會匯出 `batch.yaml` 檔案。

![fig3](https://hackmd.io/_uploads/r1Ag0_Rz-g.png)
圖 3. batch.yaml 內容範例

### 步驟 3. 提交任務列表至作業佇列

```
gsub batch.yaml
```

`gsub` 允許將任務列表提交至 GeneASIC NGSAAP 作業佇列。如果作業提交成功，將回傳一個批次 ID（Batch ID）。

### 步驟 4. 檢視作業狀態

```
gwatch
```

`gwatch` 允許檢視所有作業狀態。執行此指令後，作業佇列中的任務列表將顯示如下。要查看執行詳情，請使用方向鍵和 ENTER 鍵選擇您感興趣的特定任務。按下 ESC 鍵即可退出作業狀態檢視器。

![fig4](https://hackmd.io/_uploads/BJuFRuAzZe.png)
圖 4. GeneASIC NGSAAP 執行詳情

### 步驟 5. 檢查結果

一旦資料分析完成，您可以在步驟 2 指定的輸出目錄中檢視結果。
![fig5](https://hackmd.io/_uploads/SJRHJKCMZg.png)
圖 5. 輸出結果範例

### 2.1 自訂流程配置設定

適當的流程配置設定可帶來更好的結果，並因定序平台、資料類型、文庫製備以及速度與準確度之間的平衡等因素而異。以下是針對不同配置的幾個範例。

將 *\<identifier\>* 替換為實際的識別名稱。這些配置有助於自訂分析設定以符合特定的定序設定。`gconf` 中的 `-a` 選項會儲存配置設定，使其可在 `gsched` 中使用 `-p` 選項輕鬆重複使用。有關 `gconf` 指令的更多詳情，請參閱第 3.1 節。

  - 用於基於 PCR 的 WGS 資料

<!-- end list -->

```
gconf touch <identifier> --sequencing-protocol pcr -a
```

  - 用於 WES 資料

<!-- end list -->

```
gconf touch <identifier> --sequencing-datetype wes \
--sequencing-protocol pcr --interval-of-interest \
/opt/geneasic/share/intervals/grch38/<target-region>.bed -a
```

  - 用於 NovaSeq 或其他使用雙通道 SBS 技術的平台

<!-- end list -->

```
gconf touch <identifier> --sequencing-platform novaseq \
--polyg-trimming -a
```

  - 為了更好的準確度

<!-- end list -->

```
gconf touch <identifier> --operation-mode advanced -a
```

  - 用於從預設流程 default-germline-wes 修改配置

<!-- end list -->

```
gconf touch <identifier> -s default-germline-wes \
--interval-of-interest /path/to/target-region.bed
```

### 2.2 產生資料集列表檔案

`gsched` 指令除了像 EXCEL 這樣的傳統 CSV 編輯器外，還提供了一種高效的方式來產生用於分析的資料集列表 CSV 檔案，非常適合以相容於流程的格式管理大量樣本匯入。

  * `-s` 選項：指定 FASTQ 作為輸入
  * `-i` 選項：設定瀏覽的起始目錄，此處為 `/home/egs`
  * `-j` 選項：根據指定的正規表示式（如 `(\w+)_R[12]`）從觀察到的資料路徑中找到的第一個匹配群組，自動為每個樣本分配作業名稱。
  * `--as-csv` 旗標：將輸出格式化為 CSV

<!-- end list -->

```
gsched -s fastq -i /home/egs -j ‘(\w+)_R[12]’ --as-csv | tee data.csv
```

執行後，會出現一個文字介面供檔案選擇。使用方向鍵和空白鍵選擇樣本。應用 `tee` 指令將輸出重新導向至 `data.csv`。有關產生樣本列表檔案的更多資訊，請參閱第 3.2 節。

![fig6](https://hackmd.io/_uploads/BkMgMF0MWe.png)
圖 6. 檔案導覽與選擇範例

## 3\. 命令列操作詳情

本節提供每個指令及可用選項的詳細資訊。

### 3.1 *gconf*: 流程配置設定

`gconf` 讓使用者能夠管理資料分析流程的配置參數。不帶額外參數執行此指令將顯示所有可用的流程配置。此指令提供一系列子指令以促進流程配置的管理與自訂。

  - touch
    此子指令用於透過流程識別碼 (ID(s)) 建立或更新配置參數，提供彈性以針對不同分析階段配置各種選項。
    使用 `-s` 或 `--schema` 選項，使用者可以基於預定義的流程識別碼初始化新的配置。當啟用 `-a` 或 `--apply` 選項時，所做的任何變更將被應用並儲存至使用者的工作區。使用者自訂的設定儲存於 `/home/$USER/.ngsaap/params.yaml`。

  - inspect
    此子指令用於提供使用者特定流程配置的全面資訊。此功能使使用者能夠透過流程 ID 檢視流程配置設定，以詳細了解分析參數。

  - purge
    此子指令提供根據對應流程配置的 ID 移除配置參數的能力。此功能允許使用者管理並清理不再需要的配置。

預定義的系統級流程配置是不可變更的，**無法** 使用 `touch` 或 `purge` 子指令進行修改或刪除。

  - default-germline-wgs-pcrfree
    用於無 PCR 文庫製備的 WGS 資料的一般生殖系變異分析設定。

  - default-germline-wgs-pcr
    用於基於 PCR 文庫製備的 WGS 資料的一般生殖系變異分析設定。

  - default-germline-wes
    用於使用外顯子捕獲富集的 WES 資料的一般生殖系變異分析設定，並啟用了 PCR 偏差校正。

  - novaseq-default-germline-wgs-pcrfree
    專為使用雙通道 SBS 技術（如 Illumina NovaSeq 系列）定序的無 PCR WGS 資料設計的生殖系變異分析設定。啟用了讀段比對的 Poly-G 修剪。

  - novaseq-defualt-germline-wgs-pcr
    專為使用雙通道 SBS 技術（如 Illumina NovaSeq 系列）定序的基於 PCR 的 WGS 資料設計的生殖系變異分析設定。啟用了讀段比對的 Poly-G 修剪。

  - novaseq-defualt-germline-wes
    專為使用雙通道 SBS 技術（如 Illumina NovaSeq 系列）定序且使用外顯子捕獲富集的 WES 資料設計的生殖系變異分析設定。啟用了 PCR 偏差校正。

表 1 GeneASIC NGSAAP 全域流程配置可用選項

| 選項 | 描述 |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| \--sequencing-platform *\<CHOICE\>* | 此選項設定定序平台。*\<CHOICE\>* 可選 *mgiseq、novaseq 或 element*。預設為 *mgiseq*。 |
| \--sequencing-datatype *\<CHOICE\>* | 此選項設定定序資料類型。*\<CHOICE\>* 可選 *wgs* 或 *wes*。預設為 *wgs*。|
| \--sequencing-protocol *\<CHOICE\>* | 此選項設定定序讀段的文庫製備協定。*\<CHOICE\>* 可選 *pcrfree* 或 *pcr*。預設為 *pcr*。 |
| \--operation-mode *\<CHOICE\>* | 此選項設定分析模式。基本模式 (basic) 提供最高效能，而進階模式 (advanced) 提供較高準確度。*\<CHOICE\>* 可選 *basic* 或 *advanced*。預設為 *basic*。     |
| \--model-path *PATH* | 此選項設定外部機器學習過濾模型的路徑，將覆蓋所有流程配置。預設為 *null*。     |

表 2 GeneASIC NGSAAP 讀段比對器 (Read Mapper) 可用選項

| 選項 | 描述 |
| -------- | -------- |
| \--qual-trimming     | 此選項修剪低鹼基品質的讀段尾部。應用 --no-qual-trimming 可停用品質修剪。     |
| \--polyg-trimming     | 此選項修剪序列中的 poly-G 尾部。應用 --no-polyg-trimming 可停用 poly-G 尾部修剪。     |
| \--adaptor-trimming *\<CHOICE\>* | 此選項修剪接頭序列。*\<CHOICE\>* 可選不同的修剪方法：*hardclip*、*softclip* 和 *none*。預設為 *softclip*。     |

表 3 GeneASIC NGSAAP 變異呼叫器 (Variant Caller) 可用選項

| 選項 | 描述 |
| -------- | -------- |
| \--adaptive-pruning |此選項啟用根據觀察到的區域讀段計數自適應增加單倍體組裝的最小支援。啟用此參數可降低高覆蓋區域對定序錯誤的敏感度，提高呼叫過程的速度。|
|\--duplicate-removal|此選項移除 PCR 重複序列 (artifacts)。不建議用於無 PCR (PCR-free) 樣本。|
|\--emit-ref-confidence | 此選項發出參考信賴度分數 (reference confidence scores)。|
| \--max-reads-per-start *\<INT\>* |此選項控制每個比對起始位置保留的最大讀段數。超過此閾值的讀段將被降採樣。此值應為正整數。預設為 *4*。|
| \--interval-of-interest *PATH* |此選項參照 BED 格式的感興趣區間。它指定用於分析的基因組區域。*PATH* 應指定為 BED 檔案的絕對路徑，以便正確進行參考基因組組裝。預設為 *null*。 |

表 4 結構變異 (SV) 呼叫器可用選項

| 選項 | 描述 |
| -------- | -------- |
| \--manta-min-candidatevariant-size *\<INT\>* |此選項設定在候選發現與報告中考慮的 SV 和 indel 的最小尺寸。預設為 *8*。|
| \--manta-min-edgeobservations *\<INT\>* |此選項透過移除支援少於 *\<INT\>* 次讀數的圖形邊緣(graph edge)來進行過濾。預設為 *3*。|
|\--manta-graph-node-maxedge-count *\<INT\>*|當兩個節點的邊緣計數皆高於指定值時，此選項限制在節點之間評估的邊緣數量。預設為 *10*。|
| \--manta-min-candidatespanning-count *\<INT\>*| 此選項設定 SV 和 indel 在候選發現與報告中所需的最小跨越支援讀數。預設為 *3*。|
|\--manta-min-scored-variantsize *\<INT\>* | 此選項設定在候選識別後應評分並報告的 SV 和 indel 的最小尺寸（以鹼基對為單位）。預設為 *50*。|
|\--manta-min-diploid-variantscore *\<INT\>* | 此選項設定變異包含在二倍體 VCF 中的最小 QUAL 分數。預設為 *10*。 |
| \--manta-min-pass-diploidvariant-score *\<INT\>* | 此選項設定在二倍體 VCF 中將變異標記為已過濾的 QUAL 分數閾值。預設為 20。 |
|\--manta-min-pass-diploid-GTscore *\<INT\>* | 此選項設定在二倍體 VCF的變異項目中包含單一樣本的最小基因型品質分數。預設為 15。 |
|\--manta-min-somatic-score *\<INT\>* | 此選項設定體細胞變異包含在體細胞 VCF 中的最小品質分數。預設為 *10*。 |
| \--manta-min-pass-somaticscore *\<INT\>* | 此選項設定在體細胞 VCF 中過濾體細胞變異的品質分數閾值。預設為 *30*。 |
| \--manta-enable-remote-readretrieval-for-insertions-ingermline-calling-modes *\<INT\>* |此選項啟用在低比對品質的遠端位置檢索配對讀段，以改善生殖系插入變異的組裝。設定 *\<INT\>* 為 1 啟用，0 停用。預設為 *1*。 |
|\--manta-enable-remote-readretrieval-for-insertions-incancer-calling-modes *\<INT\>* |此選項啟用在低比對品質的遠端位置檢索配對讀段，以改善體細胞呼叫模式中插入變異的組裝。設定 *\<INT\>* 為 1 啟用，0 停用。預設為 0。|
| \--manta-use-overlap-pairevidence *\<INT\>* | 此選項定義是否應將重疊的讀段對用作證據。設定 *\<INT\>* 為 1 包含，0 排除。預設為 *0*。 |
| \--manta-enable-evidencesignal-filter *\<INT\>* | 此選項啟用針對證據訊號不顯著之候選變異的過濾器。設定 *\<INT\>* 為 1 啟用，0 停用。預設為 *1*。 |

表 5 短串聯重複序列 (STR) 大小估計可用選項
| 選項 | 描述 |
| ------ | ----------- |
| \--repeat-region-extensionlength *\<INT\>* |此選項設定目標區域周圍資訊讀段的搜尋半徑。預設為 *1000*。 |
| \--repeat-variant-catalog *PATH* |此選項設定用於變異基因分型的 JSON 目錄路徑。預設為 */opt/geneasic/database/ehunter/variant\_catalog.{hg}.json* |

表 6 HLA 分型可用選項
| 選項 | 描述 |
| ------ | ----------- |
|  \--hla-sample-population *\<CHOICE\>* | 此選項設定 HLA 分型的族群。*\<CHOICE\>* 可選 *Asian (亞洲人), Black (黑人), Caucasian (高加索人), Unknown (未知)*，以及不使用。預設為 *Unknown*。 |
| \--hla-exon-typing | 此選項啟用 HLA 的外顯子分型。 |
| \--hla-min-maf *\<FLOAT\>*| 此選項設定 HLA 分析的最小次要等位基因頻率。預設為 0.1。|

表 7 ROH 可用選項
| 選項 | 描述 |
| ------ | ----------- |
|\--roh-min-size *\<INT\>* | 此選項設定報告的最小變異尺寸（以百萬鹼基對為單位）。預設為 300。 |

表 8 結構變異註釋可用選項
| 選項 | 描述 |
| ------ | ----------- |
| \--annotsv-sv-min-size *\<INT\>*| 此選項設定用於註釋的 SV 尺寸 (bp)。預設為 *50*。 |

表 9 變異判讀報告可用選項
| 選項| 描述 |
| ------------------- | ----------- |
|\--transcript-annotation-mode *\<STR\> [STR …]* |此選項指定轉錄本註釋資料庫。*\<STR\>* 可選 Ensembl 和 Refseq。預設為 *[ Ensembl ]*。 |
| --disease-trait *\<CHOICE\>* | 此選項指定用於註釋的疾病特徵。*\<CHOICE\>* 在生殖系分析中可選 *Universal (通用)* 和 *Stroke (中風)*，在體細胞分析中可選 *Breast (乳癌), Lung (肺癌), Pancreatic (胰臟癌), Colorectal (大腸直腸癌), Chronic\_Myelogenous\_Leukemia (慢性骨髓性白血病), Cholangiocarcinoma (膽管癌)*。預設為 Universal。 |
|\--annotate-all-disease-related-genes| 此選項註釋所有與疾病相關的基因。一旦啟用此旗標，將產生名為 *\<jbname\>*\_all\_annotation.csv 的 *.xlsx* 檔案。*預設* 僅註釋致病變異。 |
|\--disease-related-gene-list *PATH* |此選項指定指定基因列表的路徑，使分析聚焦於疾病相關基因。當應用預設值 *null* 時，會參照 */opt/geneasic/tertiary/\<assembly\>/GeneList/omimGene\_list.txt*。預設為 null。 |
|\--secondary-finding-gene-list *PATH* |此選項指定次要發現基因列表的路徑。當應用預設值 null 時，會參照 */opt/geneasic/tertiary/\<assembly\>/GeneList/secondaryFinding.txt*。預設為 *null*。 |
|\--carrier-screening-gene-list *PATH*| 此選項指定帶因者篩檢基因列表的路徑。當應用預設值 null 時，會參照 */opt/geneasic/tertiary/\<assembly\>/GeneList/carrierScreening.txt*。預設為 *null*。 |
| \--clonal-gene-list *PATH* | 此選項指定株系造血基因列表的路徑。當應用預設值 null 時，會參照 */opt/geneasic/tertiary/\<assembly\>/GeneList/clonalGene\_list.txt*。預設為 *null*。 |
| \--inhouse-snv-db *PATH* |此選項指定用於 SNV 過濾的聚合內部資料庫路徑。預設為 *null*。 |
|\--inhouse-snv-cutoff *\<FLOAT\>* |此選項設定 SNV 過濾閾值，值應介於 0 與 1 之間。預設為 *0.2*。 |
|\--inhouse-sv-db *PATH* | 此選項指定用於 SV 過濾的聚合內部資料庫路徑。預設為 *null*。 |
| \--inhouse-sv-cutoff *\<FLOAT\>* |此選項設定 SV 過濾閾值，值應介於 0 與 1 之間。預設為 *0.2*。  |

### 3.2 gsched: 樣本資料配置

`gsched` 指令有助於建立用於基因組資料分析的 YAML 檔案。它使使用者能夠配置分析、選擇匯入資料、分配輸出並產生 .yaml 格式的任務列表。

使用 CSV 檔案產生任務列表 `batch.yaml` 的範例：

```
#generate a task list with a CSV file
gsched -b batch.yaml \
-p default-germline-wgs-pcrfree -s fastq -t bam vcf \
-i /home/egs - f data.csv -o results -a
```

  - **配置分析**
    允許使用選項 `-g` 選擇參考基因組版本，並使用選項 `-p` 指派流程配置

  - **匯入資料**
    指定用於分析的匯入資料目錄和/或格式

  - **分配輸出**
    指定所需的分析輸出並指派輸出目的地

  - **產生任務列表**
    建立包含所有配置的 YAML 檔案

表 10 分析配置可用選項
| 選項 | 描述 |
| ------ | ----------- |
| -g, \--assembly *\<VERSION\>* |  此選項控制用於基因組組裝的參考基因組。*\<VERSION\>* 應指定為 hs38DH、hg38、hg19 或 hs37d5。預設為 hs38d2。<br>• hs38DH<br>對應於 hs38DH，並採用適應性遮蔽方法(adapted masked approach)來處理原生參考 ALT 重疊群 (contigs)。戰略區域(Strategic regions)被遮蔽以提高準確度。<br>• hg38<br>hg38 是最新的參考基因組，對應於 hs38DH。它採用專為處理和整合原生參考 ALT 重疊群 (Alternate Loci) 而設計的適應性遮蔽方法。戰略區域被遮蔽以顯著提高比對準確度並更好地代表基因組多樣性。<br>• hs37d5<br>源自 "GRCh37" 人類基因組參考。此組裝包含誘餌序列 (decoy sequences) 以改進比對和變異呼叫。<br>• hg19<br> hg19 是之前的參考基因組。它缺乏專用的 ALT 重疊群和 hg38 相應的專門適應性遮蔽功能。其遮蔽策略較簡單，導致比對準確度較低，在高度多樣化的區域表現較不完整。|
| -p, \--pipeline *\<ID\>* | 此選項指定用於分析的流程配置。若要查看可用選項，只需在輸入 *-p 或 --pipeline* 後按兩次 TAB 鍵。或者，*gconf insepct* 指令也可以顯示所有可用的配置。|

表 11 資料匯入可用選項
| 選項 | 描述 |
| ------ | ----------- |
| -s, \--source *\<FORMAT\>* | 此選項按資料格式指定輸入來源。\<FORMAT\> 可以是以下一個或多個選擇。<br>• fastq<br> 設定 FASTQ 為雙端定序資料的輸入。檔案應分開儲存，讀段片段 1 使用如 1.fastq.gz、1.fq.gz 或 1\_001.fastq.gz 的副檔名，讀段片段 2 使用 2.fastq.gz、2.fq.gz 或 2\_001.fastq.gz。<br>• bam, vcf<br>指定已準備好分析的 BAM 和 VCF 檔案作為輸入。檔案應分別以 .bam 和 .vcf 結尾，並帶有可選的索引檔案 (.bai 或 .tbi)。<br>• sv<br>指定以 .sv.vcf.gz 結尾的已準備好分析的 SV 檔案作為輸入，帶有可選的 .tbi 索引檔案。<br>• px<br>從 .px.json 檔案載入樣本資訊。結構包含以下欄位，預設皆為空白。<br>• Order Number: 訂單的唯一識別碼。<br>• Client Order Number: 客戶分配的追蹤識別碼。<br>• Ordering Physician: 開立檢測的醫師姓名。<br>• Physician ID: 醫師的唯一識別碼。<br>• Patient Name: 患者全名。<br>• Patient ID: 患者的唯一識別碼。<br>• Birthday: 患者出生日期。<br>• Sex: 患者性別。<br>• Age: 樣本採集時的患者年齡。<br>• Sample Type: 採集的樣本類型（如血液、組織）。<br>• Sample Source: 樣本來源（如特定解剖部位）。<br>• Collect Date: 樣本採集日期。<br>• Sequencing Type: 執行的定序類型（如 WGS、WES）。<br>• Symptom: 與患者相關的症狀或狀況。<br>• Note: 與樣本相關的額外備註或說明。 |
| -i, \--import-dir *\<DIR\>* | 此選項啟用在指定目錄中進行檔案導覽以選擇分析資料。使用方向鍵在檔案和目錄間移動，按空白鍵選擇感興趣的檔案，按 ENTER 鍵停止導覽並匯出選擇。 |
|-j, \--jbname *\<REGEX\>*|此選項根據匹配檔案絕對路徑部分的特定正規表示式模式分配分組標籤。預設情況下，*\<REGEX\>* 使用模式 *([\^\\/]+)\\/[^\\/]+$*，這將父目錄名稱指派為作業名稱，非常適合每個樣本位於自己資料夾中的設置。此預設模式簡化了作業命名和分組，允許透過目錄結構進行高效的資料組織。|
| -x, \--exclude *\<DIR\>*|此選項從 *-s* 選項啟用的檔案導覽中排除不相關的檔案或目錄，從而精簡候選選擇列表。 |
| \--as-csv | 此選項連同 *-i* 和 *-s* 選項，以及可選的 *-j* 選項，產生一個 CSV 樣本列表。應用 *tee* 指令將標準輸出重新導向至副檔名為 *.csv* 的檔案。  |
| \--as-tsv | 此選項連同 *-i* 和 *-s* 選項，以及可選的 *-j* 選項，產生一個 TSV 樣本列表。應用 tee 指令將標準輸出重新導向至副檔名為 *.tsv* 的檔案。|
| -f, \--files-from *\<DIR\>* |此選項直接從 CSV 或 TSV 樣本列表匯入資料。樣本列表必須遵循以下欄位： <br>\<jbname\>,\<source-type\>/\<index\>,\<source-path\>,\<suffix-size\><br>• jbname: 特定輸入資料及後續分析的作業名稱<br>• source-type: -s 選項中允許的選擇<br>• index: 對應來源類型的特定指派<br>• source-path: 輸入檔案的位置<br>• suffix-size: gsched 產生的可選欄位，用於解決歧義<br><br>CSV 或 TSV 樣本列表可透過 *--as-csv* 或 *--as-tsv* 選項，連同 *-i* 和 *-s* 選項，以及可選的 *-j* 選項產生。|

為了準確提取群組標籤，使用者可以使用 regex101.com 等線上工具測試其正規表示式。

在圖 7 中，表示式 `(\w+)` 從每個樣本的測試字串（綠色高亮顯示）中捕獲了 sample1、sample2 和 sample3。測試時請務必在 FLAVOR 選項中選擇 "Python"。要在定案前預覽分組結果，請在執行 `gsched` 指令時啟用 `--as-csv` 或 `--as-tsv` 旗標。此預覽步驟有助於在繼續之前確保作業名稱分配的準確性和清晰度。
![fig7](https://hackmd.io/_uploads/SyHRxnCGZx.png)
圖 7. 使用線上正規表示式測試器測試群組標籤的範例

表 12 結果輸出可用選項
|  選項  | 描述         |
| ----------- | ---------------------------------- |
| -t, \--target *\<CHOICE\>* |  此選項指定所需的分析和輸出結果項目。*\<CHOICE\>* 可以是以下一個或多個選擇。<br>•fastq<br>建立指向來源 FASTQ 檔案的符號連結。<br>• bam<br>使用 GeneASIC NGSAAP 比對器執行短讀長比對，產生一個 BAM 檔案和一個副檔名為 .bai 的索引檔案。當與 -s 選項一起指定且偵測到現有 BAM 檔案時，此選項會建立指向來源 BAM 檔案的符號連結。<br>• vcf<br>使用 GeneASIC NGSAAP 呼叫器執行類似 GATK 的短變異呼叫，產生一個以 *.vcf.gz* 為副檔名的 VCF 檔案以及一個 *.vcf.gz.tbi* 副檔名的索引檔案。當與 *-s* 選項一起指定且偵測到現有 VCF 檔案時，此選項會建立指向來源 VCF 檔案的符號連結。<br>• sv<br>調用 Manta 發現結構變異，產生一個副檔名為 *.sv.vcf.gz* 的呼叫檔案和一個對應的 *.sv.vcf.gz.tbi* 檔案。如果與 -s 選項一起指定時偵測到現有的 .sv.vcf.gz 檔案，此選項將建立指向輸入檔案來源的符號連結。<br>• px<br>建立指向來源輸入 .px.json 檔案的符號連結。<br>• cnv<br>使用 ASCAT R 套件執行拷貝數變異分析，接著使用 SLMSuite 進行分割。啟用此選項會產生一個副檔名為 *.segment.SLMed.txt* 的呼叫檔案。<br>• upd<br>執行潛在單親二倍體提取，產生一個副檔名為 *.upd.txt* 的候選名單。結果用於變異判讀。<br>• smn<br>執行針對 SMN1、SMN2 以及 SMN2Δ7-8（外顯子 7-8 缺失的 SMN2）的拷貝數變異分析，產生副檔名為 *.smn.txt* 的輸出檔案。<br>• roh<br>使用 GeneASIC ROH 呼叫器執行純合子區段分析，產生副檔名為 *.roh.bed* 的輸出檔案。<br>• str<br>使用 ExpansionHunter 進行短串聯重複序列大小估計。預設變異目錄位於 */opt/geneasic/config/\<assembly\>\_variant\_catalog.json*，其中 assembly 可以是 hs38d2 或 hs37d5。結果儲存於 *.repeat.vcf* 檔案中。<br>• hla<br>使用 SpecHLA 進行人類白血球抗原分型。摘要匯出至 *.hla.txt* 檔案。<br>• mito<br>使用 GATK Mutect2 在粒線體模式下進行粒線體變異呼叫，接著使用 GATK FilterMutectCalls。輸出檔案包括一個 *.mitochondria.filtered.vcf.gz* 檔案及其伴隨的索引檔案，匯出至指定的輸出目的地。<br>• ch<br>使用 GATK Mutect2 以腫瘤僅有 (tumor-only) 模式的最佳實踐進行株系造血分析。此流程包括 GATK GetPileupSummaries、CalculateContamination 和 FilterMutectCalls，接著對可信的生殖系變異進行硬過濾。結果儲存於 *.ch.filtered.clean.vcf.gz* 檔案中，連同其索引檔案和 *.stats* 檔案。<br>• holmes<br>使用 GeneASIC Holmes 進行自動化臨床生殖系變異註釋與分類，產生 *.tsv.gz* 格式的輸出檔案。<br>• holmes\_vcf<br> 將 GeneASIC Holmes 處理的註釋和判讀附加到 VCF 檔案，產生副檔名為 *.holmes.vcf.gz* 的結果。此格式與 JBrowse 2 相容，可用於渲染 Ensembl VEP 使用的 CSQ 欄位。<br>• annotsv<br>進行結構變異註釋與排名，產生副檔名為 *.sv.annotated.tsv* 的輸出檔案。<br>• report<br>使用 ANNOVAR 以及 gnomAD、ClinVar、CADD、Revel 和 GWAS 資料庫，針對疾病相關基因、ACMG 建議的次要發現和帶因者篩檢基因區域中的呼叫變異進行變異判讀。它還包括透過 PharmCAT 進行的額外藥物基因組學臨床註釋，以及透過 PGS catalog 進行的多基因風險評分。結果彙整成 PDF 和 XLSX 檔案，如果在 gconf 指令中指定了 *--annotate-all-disease-related-genes* 旗標，則會產生可選的 \_all\_annotations.csv。<br>• report\_csv<br>匯出變異判讀產生的所有 CSV 檔案。  |
| -T, \--default-targets *\<CHOICE\>* | 為了簡化使用選項 *-t* 指定分析和輸出項目的過程，選項 *-T* 允許使用者透過單一輸入選擇多個選項。*\<CHOICE\>* 可用選項為：<br>• germline-basic<br>指定 bam, vcf, holmes 和 report 為目標<br>• germline-full<br>指定 *bam, vcf, cnv, upd, smn, str, sv, hla, roh, mito, ch, holmes, annotsv,report* 為目標 |
| -o, \--export-dir \<DIR\>  |   此選項指定輸出目錄。若未指定目錄，預設為當前目錄。 |
|  \--group-by-jbname  |    此選項根據作業名稱將結果組織到單獨的資料夾中。   |
|  \--flatten-directory-structure  | 此選項將巢狀目錄扁平化，將所有輸出檔案儲存在單一目錄中。  |

表 13 任務列表產生可用選項
| 選項 | 描述 |
| ------ | ----------- |
| -b *PATH* | 此選項指定 YAML 格式的批次配置檔案路徑。如果未提供路徑，任務列表將以 batch.yaml 儲存在當前目錄下。可以多次覆寫 YAML 檔案，以便在單一檔案中配置具有不同流程設定的樣本。 |
| \--jbname-suffix | 此選項將後綴字串附加到衝突的作業名稱，以解決與作業名稱衝突相關的任何歧義。 |
|-a, \--apply | 此選項允許退出預覽模式並儲存配置。 |
| \--overwrite | 此選項替換現有的作業名稱。 |
| \--skip |  此選項跳過名稱衝突的作業。 |

### 3.3 gsub: 作業提交

`gsub` 指令允許使用者將使用 `gsched` 指令建立的 `batch.yaml` 檔案中指定的任務提交至 GeneASIC NGSAAP 作業佇列。要啟動樣本分析任務的提交，請執行以下指令：

```
gsub batch.yaml
```

執行後，此指令將為您的提交提供一個唯一的批次 ID，建立 `batch.yaml` 檔案中定義的相應檔案和目錄，並在當前工作目錄中產生記錄檔 (log files)。要監控作業狀態並存取詳細的執行資訊，請參閱第 3.4 節關於使用 *gwatch* 指令的指南。

表 14 作業提交可用選項
| 選項 | 描述 |
| ------ | ----------- |
| -a, \--array *\<STR\>* | 此選項允許使用者使用從 1 開始的陣列索引值選擇感興趣的樣本進行提交。選項參數可以是逗號分隔的值、連續的索引值範圍，以及可選的步長，如下例所示：<br>• -a 1-10<br>提交索引值在 1 到 10 之間的作業陣列<br>• -a 1,3,5,7<br>提交索引值為 1, 3, 5 和 7 的作業陣列<br>• -a 1-7:2 提交索引值在 1 到 7 之間且步長為 2 的作業陣列。預設設定將 `batch.yaml` 中定義的所有任務提交至 GeneASIC NGSAAP 作業佇列。 |
| -w, \--nodelist *\<STR\>* | 此選項允許使用者以逗號分隔字串格式指定特定的運算主機列表。分析作業將根據需要排程在這些指定的主機上，以滿足資源需求。列表中的任何重複節點名稱將被忽略，確保每個節點僅計算一次。 |
|-x, \--exclude *\<STR\>* | 此選項允許使用者指定應從排程過程中排除的主機列表。它確保某些節點不用於作業，從而允許對分析流程中的資源分配進行更多控制。 |

### 3.4 gwatch: 作業佇列檢視器

`gwatch` 指令使使用者能夠輕鬆監控作業狀態並存取系統內的詳細執行資訊。透過執行不帶任何額外參數的 `gwatch` 指令，使用者可以啟動文字介面的作業狀態檢視器。

在檢視器中，每個執行記錄被歸類為「進行中 (In Progress)」或「已完成 (Finished)」。使用者可以選擇以簡潔或詳細模式檢視這些記錄，並使用方向鍵和空白鍵方便地在兩者之間切換。

在簡潔模式中，每個記錄由呈現為「BATCH ID\_TASK ID」的作業 ID 唯一標識。此外，每個任務都與由 `--gsched` 指令解析的作業名稱相關聯，並標記為以下狀態之一以指示其進度：

  - RUNNING (執行中)：表示正在積極執行的任務。
  - PENDING (等待中)：表示在佇列中等待可用資源開始的任務。
  - COMPLETE (完成)：代表已成功完成執行的任務。
  - FAILED (失敗)：標記在執行過程中遇到錯誤或問題且未按預期完成的任務。
  - CANCELLED (取消)：識別在完成前被故意終止的任務。

此外，簡潔模式中的每個記錄都會顯示作業的經過時間，為使用者提供任務執行持續時間的快速概覽。

在詳細模式中，使用者不僅可以存取關於作業所有權、運算節點名稱、提交、開始和結束時間點的資訊，還可以存取流程階段級別的執行資訊。

預設情況下，`gwatch` 檢索 4 小時窗口內的作業執行歷史記錄。但是，使用者可以使用 `--within` 選項指定所需的歷史窗口，例如 `--within 8` 以查看過去 8 小時內的作業執行歷史記錄。

### 3.5 gcancel : 作業取消

`gcancel` 指令用於終止受 GeneASIC NGSAAP 控制的作業。

表 15 作業終止可用選項
| 選項 | 描述 |
| ------ | ----------- |
| \--batch *\<ID\> [ID ...]* | 此選項允許使用者透過指定一個或多個批次 ID 來終止一批任務。 |
| \--jobid *\<ID\> [ID ...]* |此選項允許使用者透過指定一個或多個作業 ID 來終止任務。 |
|\--user *\<STR\>* | 此選項將取消操作限制為特定使用者擁有的作業。 |
| \--me | 此選項將取消操作限制為當前使用者擁有的作業。 |

### 3.6 gversion : 程式版本檢視器

`gversion` 指令顯示 GeneASIC NGSAAP 的全面版本資訊，包括硬體、韌體、命令列工具、第三方函式庫和基因組資料庫版本。

圖 8. GeneASIC NGSAAP 版本資訊範例

### 3.7 acctmgr : 使用者帳號管理員

GeneASIC 帳號管理員 (acctmgr) 是一個命令列實用程式，用於管理 NGSAAP 環境中的使用者帳號和群組。

⚠ **權限與安全**
在使用這些指令之前，請確保您擁有必要的管理權限。
**警告：** 刪除使用者或群組是 **不可逆** 的動作。請務必極其謹慎地進行。

**指令參考**

  - acctmgr list
    列出系統中所有現有的使用者帳號和群組，提供當前配置的完整概覽。
  - acctmgr +g \<groupname\> [\<gid\>]
    建立一個新的使用者群組。如果未指定可選的群組 ID (\<gid\>)，系統將自動分配一個。
  - acctmgr +u \<username\> \<groupname\> [\<uid\>]
    將新使用者新增至主要群組。可以提供可選的使用者 ID (\<uid\>)；否則，將自動分配一個。建立後，系統將提示您設定新使用者的密碼。
  - acctmgr +G \<username\> \<groupname\>
    將現有使用者新增至次要群組。這通常用於授予額外權限。
    o **範例：** `acctmgr +G john fpgauser` 將使用者 john 新增至 fpgauser 群組，授予他們使用 FPGA 分析基因組資料的權限。
  - acctmgr -u \<username\>
    從系統中永久刪除使用者。
  - acctmgr -g \<groupname\>
    從系統中永久刪除使用者群組。
  - acctmgr -G \<username\> \<groupname\>
    從指定的次要群組中移除使用者，而不刪除使用者的帳號。

## 4\. 修訂歷史

| 版本 | 日期 | 變更 |
| ------ | --- | ----------- |
| 1.1.0\_b | 2025 Dec | 小幅更新 |
| 1.1.0\_a| 2024 Nov |  更新 GeneASIC NGSAAP All-in-One 版本的完整功能  |
| 1.0.0  | 2023 Mar|  初始版本   |