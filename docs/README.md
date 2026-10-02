# 文件

concept-rule-attribution 的設計規格與決策紀錄。

根目錄的 [README](../README.md) 說明專案是什麼、為什麼使用格子世界，以及目前所在階段。這裡的文件說明每個部分如何運作，細節足以直接依此實作，不必回頭重讀原始討論。

## 結構

| 路徑 | 內容 | 何時閱讀 |
| --- | --- | --- |
| `design/` | 現行規格：每個元件做什麼、每個量如何定義 | 實作或修改元件時 |
| `decisions/` | 決策紀錄：為什麼這樣選、放棄了什麼、代價是什麼 | 想知道某件事為什麼是現在這樣時 |

`design/` 只描述現行設計。設計改變時直接改寫文件，改變的理由寫進新的決策紀錄。決策紀錄接受後不再修改。

## 設計文件

| 文件 | 範圍 |
| --- | --- |
| [environment.md](design/environment.md) | MiniGrid 的使用方式、客製環境必須具備的要素、反事實標準答案的取得方式、資料與硬體 |
| [world-model.md](design/world-model.md) | 編碼器與預測器的必要條件、「什麼都不做」動作的輸入方式、差異量的候選 |
| [rule-extraction.md](design/rule-extraction.md) | 從表示空間抽出規則的流程、規則形式與接受標準 |
| [attribution.md](design/attribution.md) | 兩種歸因估計如何計算、如何與標準答案比對，以及規則解釋不了的變化如何處理 |
| [evaluation.md](design/evaluation.md) | 第零到第四階段的方法與通過條件，以及所有階段共用的對照方法 |
| [comparisons.md](design/comparisons.md) | 第四階段比較的各組設定，以及每一組排除了哪一種解釋 |

## 決策紀錄

[decisions/](decisions/README.md) 存放編號的紀錄。每筆紀錄自成一體，自行重述需要的背景，不連結到日後可能被改寫的規格。

## 閱讀順序

第一次閱讀：先讀根目錄 README，再讀 `design/environment.md` 了解代理實際看到與能做的事，接著讀 `design/evaluation.md`，因為評估方式約束了其他所有設計，其餘設計文件順序不拘。
