# 文件

concept-rule-attribution 的設計規格、決策紀錄、計畫與參考資料。

根目錄的 [README](../README.md) 說明專案是什麼、為什麼使用格子世界，以及目前所在階段。這裡的文件說明每個部分如何運作，細節足以直接依此實作，不必回頭重讀原始討論。

## 結構

| 路徑 | 內容 | 更新規則 | 何時閱讀 |
| --- | --- | --- | --- |
| `design/` | 現行規格：每個元件做什麼、每個量如何定義、如何判定 | 設計改變時直接改寫，只描述現況 | 實作或修改元件時 |
| `decisions/` | 決策紀錄：為什麼這樣選、放棄了什麼、代價是什麼；也用來鎖定各階段的門檻 | 接受後不再修改 | 想知道某件事為什麼是現在這樣時 |
| `plan/` | 路線圖與風險：要做哪些工作、順序、目前進度、可能出錯的地方 | 進度或狀況改變時直接更新 | 安排工作或回報進度時 |
| `reports/` | 各階段的結果報告。第一份報告完成時建立 | 完成後不再修改，更正另寫新報告 | 想知道某項檢驗的結果時 |
| `glossary.md` | 術語表：中文術語、英文術語與程式碼名稱的對應 | 新增術語時更新 | 撰寫文件或程式碼時 |
| `related-work.md` | 相關研究與本專案的定位 | 每個里程碑結束時檢查 | 想知道本專案與既有研究的關係時 |

`design/` 只描述現行設計。設計改變時直接改寫文件，改變的理由寫進新的決策紀錄。決策紀錄接受後不再修改。

`plan/` 描述工作，不描述設計。檢驗的方法與通過條件只寫在 `design/`，`plan/` 引用它們。

## 設計文件

| 文件 | 範圍 |
| --- | --- |
| [environment.md](design/environment.md) | 客製環境的物體、規則表、機制與環境變體、帶雜訊的變體、可識別性、標準答案與反事實、環境必須滿足的性質 |
| [data.md](design/data.md) | 行為策略、覆蓋率檢查、資料分割、每一步保存的內容、儲存格式 |
| [world-model.md](design/world-model.md) | 編碼器、信念模組與轉移頭的必要條件，差異量，診斷 |
| [readout.md](design/readout.md) | 讀取目標、對照、單張畫面上限、指標、介入檢查，以及把表示讀成狀態 |
| [rule-extraction.md](design/rule-extraction.md) | 從讀出的狀態抽出規則的流程、規則形式、搜尋方法、描述長度、保真度–複雜度曲線與判定 |
| [attribution.md](design/attribution.md) | 歸因的定義、三種估計與基準、分層、指標、規則解釋不了的變化 |
| [evaluation.md](design/evaluation.md) | 各階段的方法、通過條件與相依關係 |
| [comparisons.md](design/comparisons.md) | 第四階段比較的各組設定，以及每一組排除了哪一種解釋 |
| [statistics.md](design/statistics.md) | 種子、區間估計、校準、多重比較、門檻鎖定程序與報告格式 |
| [architecture.md](design/architecture.md) | 程式碼的目錄、模組界線、執行紀錄、測試與釋出 |

## 決策紀錄

[decisions/](decisions/README.md) 存放編號的紀錄。每筆紀錄自成一體，自行重述需要的背景，不連結到日後可能被改寫的規格。

## 閱讀順序

第一次閱讀：

1. 根目錄 README。
2. [design/environment.md](design/environment.md)：代理實際看到與能做的事，以及世界的規則。
3. [design/evaluation.md](design/evaluation.md)：評估方式約束了其他所有設計。
4. 其餘設計文件，順序不拘；遇到不熟的術語查 [glossary.md](glossary.md)。
5. [plan/roadmap.md](plan/roadmap.md)：目前進度與下一步。

## 結構的依據

- 規格與決策紀錄分開，決策紀錄採用 Nygard（2011）提出的格式。
- 依讀者的需求區分文件種類，參考 Diátaxis：`design/` 與 `glossary.md` 是查閱用的參考，`decisions/` 與 `related-work.md` 是說明為什麼，`plan/` 是工作安排。
- 門檻鎖定與結果報告的程序，參考機器學習的可重現性清單（Pineau 等，2021）與事先登記的做法（Nosek 等，2018）。

完整的參考文獻在 [related-work.md](related-work.md)。
