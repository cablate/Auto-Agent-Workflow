# Gemini 自動化內容產出工作流

## 項目簡介

這個工作流旨在為內容創作者提供一個標準化、高效率、且高度客製化的文章產出流程。透過結合 AI 的強大生成與分析能力，並運用您獨特的文筆風格，我們能夠將原始素材或主題，轉化為一篇篇具有深度與個人特色的文章。整個流程包含素材研究、風格學習、多輪草稿撰寫與審查，確保最終產出文章的品質與連貫性。

## 核心設計理念

本工作流的核心，是將複雜的文章產出過程拆解為 10 個清晰的步驟，並為每個步驟配備了專屬的 AI Agent Prompt。您（使用者）將扮演『**流程協調者 (Orchestrator)**』的角色，透過 AI 自動逐步執行每個任務，並審查中間產出，確保流程按照您的預期推進。

---

## 使用須知

1. 強烈建議您使用低成本的模型應用，例如 Gemini Cli (gemini-2.5-flash)，目前 flash 每日有 1000 次請求額度，而每一次的文章任務大約會使用 15-20 次不等。
2. 在準備您個人歷史創作集，建議將數量控制在一個範圍，如果是 200-300 字的創作，那麼約 20-30 篇就已經可以有很好的效果。
3. 如果您有在進行探討與解析自我，也建議您一併提供，這將有助於 AI 從你的價值觀出發，而不是純粹地模仿。
4. 如果中途 AI 斷掉 (等待您的提問)，直接回覆 `繼續` `Continue` 即可。

## 如何開始?

您可以選擇 `git clone` 或直接下載並解壓縮此專案，並於這個目錄啟用 `gemini --yolo -m "gemini-2.5-flash"` <br/>
請注意，我們強烈不建議您使用寶貴的 `gemini-2.5-pro` 額度，但如果您沒有使用 pro 的習慣那麼也可以考慮。 <br/>

您可以先將一些資訊寫入 md 並存放於根目錄 `ref.md`，並輸入以下提問啟動流程:
``` bash
# 其中最後的 "我希望xxx" 可以替換為任意需求
@ref.md 請你將這份文件根據我們 workflow.yml 的流程完整走一次。我希望可以擴寫更多內容。
```

## 事先警告

由於本專案是 Agent 驅動，而不是以程式控制 workflow，模型具有隨機性，表現不符合預期是正常的，請確實下好提問、準備好您的參數設定。<br/>
您可能會面臨一次任務需要手動介入的情況，您可以請 AI 從意外中斷或是您認為執行不夠好的步驟開始接續執行。

---

## 工作流程概覽

整個流程分拆為以下 10 個核心步驟：

1.  **設定讀取與環境準備**
2.  **生成待辦事項清單 (`.outputs/{run_name}/todo.md`)**
3.  **處理輸入並產出原始資料 (`.outputs/{run_name}/original-data.md`)**
4.  **審查原始資料 (`.outputs/{run_name}/reviews/review_original_data.md`)**
5.  **學習作者文筆風格 (生成或讀取 `.memory/author-style-methodology.md`)**
6.  **撰寫文章初稿 (`.outputs/{run_name}/drafts/draft-1.md`)**
7.  **審查文章初稿 (`.outputs/{run_name}/reviews/review_draft_1.md`)**
8.  **根據審查結果進行二修 (`.outputs/{run_name}/drafts/draft-2.md`)**
9.  **進行二次審查 (`.outputs/{run_name}/reviews/review_draft_2.md`)**
10. **產出最終完稿 (`.outputs/{run_name}/final/final-article.md`)**

---

## 如何開始使用

### 1. 配置 `workflow.yml`：任務的藍圖

`workflow.yml` 是整個工作流的中心配置文件。在開始任何新任務之前，您都需要根據本次任務的需求仔細配置此檔案。

#### 關鍵配置項說明：

*   **`run_name`**：
    *   **用途**：定義本次執行任務的名稱。這個名稱將被用來建立一個專屬的資料夾 (`.outputs/{run_name}/`)，所有本次任務的暫存檔案都將存放在這個資料夾中。請確保每次新任務都使用獨一無二的名稱，以避免檔案衝突。
    *   **範例**：`run_name: "關於規格文件的重要性"`

*   **`input_processing`**：
    *   **用途**：定義文章的原始輸入來源與處理方式，這將影響 `original-data.md` 的內容。<br>
    *   **`type`**：
        *   `"topic"`：輸入是一個文字主題字串，AI 將根據此主題進行研究並擴展內容。
        *   `"url"`：輸入是一個網址，AI 將抓取並分析網頁內容。
        *   `"article_file"`：輸入是一個區內部檔案路徑，AI 將讀取該檔案內容。
    *   **`source`**：根據 `type` 的選擇，填寫主題字串、URL 或檔案路徑。<br>
    *   **`market_research_required`**：
        *   `true`：即使 `source` 已提供，AI 也會進行背景研究並適度擴寫。這適用於 `source` 僅為一個簡短想法或大綱的情況。
        *   `false`：AI 主要會依據 `source` 內容進行提煉與結構化，不做額外擴寫。適用於 `source` 已是完整文章或不想以研究方式擴寫的情況。
    *   **範例**：
        ```yaml
        input_processing:
          type: "topic"
          source: "AI 如何優化軟體開發的工作流程"
          market_research_required: true
        ```
        ```yaml
        input_processing:
          type: "article_file"
          source: "關於規格.md"
          market_research_required: false # 即使是檔案，如果內容簡短想進行擴寫可以在提問的時候提出要求
        ```

*   **`style_learning.force_refresh`**：
    *   **用途**：控制 `author-style-methodology.md` (您的文筆風格指南) 的生成行為。
    *   `true`：強制 AI 重新分析 `style_learning.generation.source_files` 中的所有過往個人創作集，並更新風格指南，這一步可能花費5~10分鐘。<br>
    *   `false`：如果風格指南檔案已存在 (`.memory/author-style-methodology.`)，AI 將跳過重新分析，直接使用現有的指南。這能大幅節省 Token 並加速流程。<br>

*   **`style_learning.source_files`**：
    *   **用途**：控制參考創作集的讀取範圍與路徑，可以使用正規表達式，或簡單以 `_learning_datas/**/*` 表示讀取所有底下的檔案。

### 2. 執行流程：與 AI 協同作業

由於我們將『流程控制』交由您（協調者）來主導，您並不需要一步步地下達指令，

#### 範例指令格式：

當您完成 `workflow.yml` 的配置後，您可以下達類似以下的指令啟動流程：

> `請你根據我們workflow.yml的流程，為此次任務生成待辦事項清單。 @file.md or 主題:自動化 or 新聞連結，意圖是想寫一篇社群貼文`

之後每當 AI 完成一個步驟的產出，AI 便會自動推進到下一個排程中的步驟，並執行對應的 Prompt。<br>

### 3. 如何處理輸入 (針對 `original-data.md` 的產出)

*   **要點**：確保 `workflow.yml` 中 `input_processing` 的 `type`、`source`、`market_research_required` 配置清晰，讓 AI 能正確理解您的意圖。
*   **範例**：
    *   **Case A: 給定簡短想法，要求 AI 大幅擴充 (例如：`關於規格.md` 內容只有幾句話)**
        ```yaml
        input_processing:
          type: "article_file"
          source: "關於規格.md"
          market_research_required: true # <-- 設為 true 才會觸發擴展
        ```
    *   **Case B: 給定全面主題，要求 AI 研究並歸納**
        ```yaml
        input_processing:
          type: "topic"
          source: "AI 在軟體開發安全測試的最新應用與趨勢"
          market_research_required: true
        ```
    *   **Case C: 給定一篇現有文章/URL，要求 AI 分析提煉**
        ```yaml
        input_processing:
          type: "url"
          source: "https://example.com/a-long-article-about-ai"
          market_research_required: false # <-- 設為 false 不會進行主題研究
        ```

---

## 產出檔案結構

所有流程產出的檔案都將存放在 `.outputs/{run_name}/` 資料夾中。

最終的成品會存放在 `.complete` 資料夾中

```
.complete/                            # 文章成品
.outputs/
└── {run_name}/                       # 您的本次任務名稱
    ├── todo.md                       # 本次任務的待辦清單
    ├── original-data.md              # 原始資料/研究報告
    ├── draft-1.md              
    └── draft-2.md
```

---

## 重要提示

*   **協調者的角色**：請記住，您是整個流程的『協調者』。AI 會根據 `workflow.yml` 和具體的 Prompt 執行任務，但需要在一開始都由您來掌控所有的參數設定、以及準確的提問指令讓 workflow 可以正常執行。
*   **文筆風格指南 (`.memory/author-style-methodology.md`)**：這份檔案是 AI 學習您文筆的核心。若您文筆有顯著變化或有大量新文章，可以將 `workflow.yml` 中的 `style_learning.force_refresh` 設為 `true`，或直接刪除該檔案，以強制 AI 重新分析並更新此指南。

---

## 最後

祝您在 AI 輔助創作有美好的體驗。