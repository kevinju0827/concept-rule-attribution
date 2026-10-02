# 術語表

文件使用的中文術語、對應的英文術語，以及程式碼中的名稱。撰寫文件與程式碼時以本表為準；新增術語時一併更新。

## 模型與表示

| 術語 | 英文 | 程式碼 | 說明 |
| --- | --- | --- | --- |
| 世界模型 | world model | `models` | 在表示空間中預測下一步的模型 |
| 表示 | representation | | 模型把畫面壓縮後得到的內部數值 |
| 表示空間 | representation space, latent space | | 所有表示構成的空間 |
| 編碼器 | encoder | `encode` | 一張畫面到單張畫面表示 |
| 單張畫面表示 | frame embedding | `z` | 只依賴一張畫面的表示，記為 `z_t` |
| 信念模組 | belief module | `belief` | 從歷史產生歷史條件表示 |
| 歷史條件表示 | belief representation | `b` | 只依賴第 t 步以前的畫面與第 t−1 步以前的動作，記為 `b_t` |
| 轉移頭 | transition head | `transition` | 從 `b_t` 與動作預測 `ẑ_{t+1}` |
| 表示崩塌 | representation collapse | | 所有輸入被壓成幾乎相同的表示 |
| 有效秩 | effective rank | | 監看崩塌的指標 |

## 環境

| 術語 | 英文 | 程式碼 | 說明 |
| --- | --- | --- | --- |
| 客製環境 | custom environment | `PushRoomEnv` | 本專案自建的 MiniGrid 環境 |
| 狀態 | state | `State` | 環境的完整結構化狀態，含隱藏變數 |
| 標準答案 | ground truth | | 從環境內部讀出的 `State` 或其差異 |
| 什麼都不做 | no-op | `noop` | 代理不施加任何效果，環境照常運作 |
| 作用 | toggle | `toggle` | 對前方物體的動作；對箱子時為擊中 |
| 推動冷卻 | push cooldown | `cooldown` | 可切換的看不見的時間規則 |
| 規則開關 | rule switch | `push_cooldown` | 冷卻步數；0 表示關閉 |
| 隱藏變數 | hidden variable | | 不畫在畫面上的變數；分為自身與他者 |
| 動態變數 | dynamic variable | | 單張畫面看不到、連續畫面可推得的變數，例如球的行進方向 |
| 受擊提示 | hit cue | `hit_cue` | 箱子被擊中後只持續一張畫面的外觀 |
| 可識別 | identifiable | `identifiable` | 隱藏變數此刻的值已由觀測歷史唯一決定 |
| 記憶年齡 | memory age | `memory_age` | 決定隱藏變數目前值的事件距今幾步 |
| 預防事件 | prevented event | `prevented` | 什麼都不做時會發生、實際上因代理而沒發生的事件 |
| 促成事件 | enabled event | `enabled` | 實際上因代理才發生的環境事件 |
| 行為策略 | behavior policy | | 產生離線紀錄時選擇動作的方式 |
| 覆蓋率檢查 | coverage check | | 確認每條規則與每個值在資料中出現足夠多次 |

## 讀取

| 術語 | 英文 | 程式碼 | 說明 |
| --- | --- | --- | --- |
| 讀取器 | probe | `readout` | 從表示預測真實變數的模型 |
| 線性讀取器 | linear probe | | 只做加權相加的讀取器，本專案確認性分析唯一使用的形式 |
| 讀取目標 | probe target | | 讀取器要預測的變數 |
| 相對代理的目標 | agent-relative target | `front_type` 等 | 以代理為參考點定義的變數 |
| 原始像素參考 | pixel reference | | 直接在像素上的線性讀取，說明變數是否本來就能從輸入讀出 |
| 單張畫面上限 | single-frame ceiling | | 以真實的可見狀態預測隱藏變數的最佳準確率 |
| 洩漏檢查 | leak check | | 確認隱藏變數不會從單張畫面被讀出超過上限 |
| 描述長度 | minimum description length | `mdl` | 讀取器學會標籤需要多少資料；以線上編碼計算 |
| 介入檢查 | intervention | | 沿著讀取器的方向修改表示，檢查預測是否照預期改變 |

## 規則

| 術語 | 英文 | 程式碼 | 說明 |
| --- | --- | --- | --- |
| 規則抽取 | rule extraction | `rules` | 從讀出的狀態搜尋規則 |
| 規則 | rule | | 「條件 → 效果」 |
| 環境規則 | environment rule | `env_phase` | 從什麼都不做的轉移抽出的規則 |
| 動作規則 | action rule | `agent_phase` | 描述各動作效果的規則 |
| 狀態轉移函式 | transition program | `step` | 抽出的規則組成的可執行 Python 函式 |
| 容忍比例 | tolerance | `q` | 搜尋時規則須重現的最低比例 |
| 上限對照 | upper bound | | 在真實狀態上執行相同的搜尋 |
| 歷史特徵 | history feature | | 由讀出的狀態序列推得的特徵，例如距上次成功推動的步數 |

## 歸因

| 術語 | 英文 | 程式碼 | 說明 |
| --- | --- | --- | --- |
| 歸因 | attribution | `attribution` | 某一步的變化中由代理造成的部分 |
| 對照 | contrast | | 歸因以什麼都不做為對照 |
| 實際分支 | actual branch | | 執行實際動作的分支 |
| 不作為分支 | no-op branch | | 執行什麼都不做的分支 |
| 神經網路歸因 | model-based attribution | `nn` | 轉移頭在兩個分支下的預測之差 |
| 規則歸因 | rule-based attribution | `rule` | 狀態轉移函式在兩個分支下的模擬之差 |
| 差異量 | difference measure | | 兩個預測之間的距離 |
| 一步歸因、H 步歸因 | one-step, H-step attribution | `horizon` | 比較的時間範圍 |
| 分層 | stratum | | 依標準答案與事件對步驟的分類 |
| 剩餘變化 | residual | | 觀察到的變化扣除規則能解釋的部分 |
| 無法歸因 | unattributable | | 剩餘變化中無法判定來源的部分 |
| 有特權、沒有特權 | privileged, non-privileged | | 剩餘變化的判定是否使用複製環境的能力 |
| 三方比對 | three-way comparison | | 神經網路歸因、規則歸因與標準答案的比對 |

## 評估

| 術語 | 英文 | 程式碼 | 說明 |
| --- | --- | --- | --- |
| 基準線 | baseline | | 相同架構、未經訓練的模型的結果 |
| 試驗 | pilot | `pilot` | 用來設定門檻的執行與資料，不用於確認性結果 |
| 確認性 | confirmatory | | 門檻鎖定紀錄中事先列出的分析 |
| 探索性 | exploratory | | 其餘所有分析 |
| 門檻鎖定 | pre-registration | `prereg/stage-N` | 以決策紀錄與 git 標籤固定門檻 |
| 分級推廣測試 | graded out-of-distribution test | `ood` | 依偏離訓練分布的程度排序的測試 |
| 保留比例 | retention | | 推廣測試中第一、二級的平均除以第零級 |
| 同閘控的雙網路 | gated dual network | | 推廣測試的對照設定 4a |
| 停止規則 | stopping rule | | 未通過的檢驗擋住依賴它的檢驗，不以擴大規模繞過 |

## 已停用的術語

| 術語 | 停用原因 |
| --- | --- |
| 選擇性差距 | 原本指以打亂標籤訓練的讀取器與正常讀取器之間的準確率差距；在保留資料上它只等於準確率減去隨機猜測，理由見 [decisions/0009](decisions/0009-probe-targets-and-controls.md) |
