# concept-rule-attribution

在規則完全已知的格子世界中檢驗兩件事：從畫面學出的表示空間，能否抽出精確、可驗證的規則；以及代理能否估計環境變化中有多少是自己造成的。

規格放在 [docs/](docs/README.md)。本文件說明專案是什麼、為什麼要做；各部分如何運作，寫在 docs 之下的文件裡。

## 背景

目前的大型語言模型幾乎只從文字學習。文字不是世界的原始資料，而是人類已經做出的結論的壓縮紀錄，所以模型讀文字，等於直接繼承了寫在符號裡的因果結構，而不是自己發現它。詞與概念的對應也不完美：一字多義、一義多字，有些概念甚至沒有對應的詞。以這樣的資料訓練，很難得到精確的概念與邏輯能力。

神經科學的證據指向同一個方向。大腦的語言網路和負責數學、邏輯推理的網路是分開的，嚴重失語症的病人仍能解數學題與推理（Fedorenko 等，2024）。主動介入也是被動觀察無法取代的：兩隻小貓接收幾乎相同的視覺輸入，只有主動移動的那隻發展出正常的視覺引導行為（Held 與 Hein，1963）。

近年已有幾條研究路線分別處理其中一部分：

- 在表示空間中預測世界的模型。表示空間指模型把畫面壓縮後得到的內部數值。代表為 JEPA 系列，例如 V-JEPA 2 與 LeWorldModel。
- 不經過文字的推理，例如 Coconut 及其後續改良。
- 預測概念向量、需要時才轉回文字，例如 VL-JEPA。
- 以程式碼表示環境規則的世界模型，例如 OneLife 與 OPINE-World。
- 在世界模型中區分代理造成的變化與環境自己的變化，例如以條件互資訊偵測代理影響的 CAI，以及以空動作分離世界效果與動作效果的 DWM。

目前仍未被接上的缺口是：連續的表示空間與精確的規則之間，沒有不經過語言的橋。現有的串接方式，幾乎都是讓大型語言模型在中間用文字轉譯。而代理對自身影響的估計，也還沒有在一個能取得精確反事實答案、包含隱藏變數的環境中被逐項驗證過。

各研究線與本專案的異同，整理在 [docs/related-work.md](docs/related-work.md)。

## 為什麼使用格子世界

本專案在一個所有變數都可控、標準答案可直接取得的格子世界中，驗證自身貢獻歸因的核心機制，並同時進行規則抽取。核心機制若在這裡不成立，更複雜的環境也不會讓它成立；若成立，之後要回答的問題就收斂成「這個機制在真實的多來源混淆下是否仍然有效」。

同時研究兩個目標的理由與取捨記錄在 [decisions/0001](docs/decisions/0001-combine-rule-extraction-and-attribution.md)。

## 研究問題

1. **概念。** 不經文字、直接從畫面學出的表示空間，是否形成對應真實狀態的概念，包括看不見的隱藏變數，以及屬於代理自身與屬於其他物體的變數？
2. **規則。** 動作在表示空間中是否對應一致的轉換？能否不經大型語言模型，直接抽出精確、可驗證的規則？抽出的規則是否比神經網路更能推廣到沒看過的情況？
3. **歸因。** 神經網路的反事實歸因估計、以規則模擬得到的歸因估計，以及環境提供的標準答案，三者是否一致？

## 兩個目標的連結

如果能抽出精確規則，就能把規則分成兩類：「什麼都不做」時環境自己如何變化，以及每個動作如何改變狀態。用規則分別模擬「實際做的動作」和「什麼都不做」，兩者的差就是代理自身的影響。這和神經網路的反事實歸因回答的是同一個問題，只是以可讀、可檢查的形式回答。

規則解釋不了的變化，不能直接當作代理自己造成的。它混合了三種來源：沒抽完整的動作規則、規則沒涵蓋的其他來源，以及規則本身的誤差。區分它們的方法，是主動改變自己的動作，看剩餘的變化是否跟著改變。

設計細節在 [design/attribution.md](docs/design/attribution.md)，決策理由在 [decisions/0004](docs/decisions/0004-rule-simulation-as-second-attribution-estimate.md) 與 [decisions/0006](docs/decisions/0006-noop-semantics-and-contrastive-attribution.md)。

## 環境

以 MiniGrid 自建的 8×8 格子世界：代理可以推動箱子、擊中箱子、走上充電格；巡邏球依固定規則來回移動。箱子的耐久度、代理的能量與推動後的冷卻都不畫在畫面上，只能從歷史推得。推動冷卻可以單獨開關，作為成對比較的對象。所有轉移都是確定性的，複製環境即可得到任何動作的精確反事實結果。

規則表與設計在 [design/environment.md](docs/design/environment.md)，取捨在 [decisions/0005](docs/decisions/0005-deterministic-custom-environment.md)。

## 階段

各項檢驗依相依關係推進，每一項都有事先鎖定的通過條件；未通過的檢驗只擋住依賴它的檢驗，且不以擴大規模繞過。

| 階段 | 內容 | 回答的問題 |
| --- | --- | --- |
| 0 | 重現世界模型的公開結果；驗收客製環境與資料 | 工具與環境是否可用 |
| 1 | 訓練世界模型，檢查表示是否記錄了真實狀態，包括隱藏變數 | 概念是否形成 |
| 歸因檢驗 | 比較神經網路的反事實歸因與標準答案；只依賴第一階段 | 核心機制是否成立 |
| 2 | 檢查每種動作在表示空間中造成的變化 | 動作是否對應一致的轉換 |
| 3 | 從表示抽出規則，以真實狀態判定是否正確 | 能否抽出精確規則 |
| 4 | 推廣測試、規則開關的成對比較、三方歸因比對 | 規則是否推廣得更好，歸因是否一致 |

各階段的方法與通過條件在 [design/evaluation.md](docs/design/evaluation.md)，統計方法與門檻鎖定程序在 [design/statistics.md](docs/design/statistics.md)。

## 狀態

設計文件完成，尚未撰寫任何程式碼。工作安排與目前進度在 [plan/roadmap.md](docs/plan/roadmap.md)，風險在 [plan/risks.md](docs/plan/risks.md)。

## 下一步

1. 建立程式碼骨架與持續整合（M0）。
2. 在目標 GPU 上量測 LeWorldModel 的顯示記憶體用量並重現一個公開環境，決定 [decisions/0003](docs/decisions/0003-lewm-as-world-model-starting-point.md) 是否接受（M1）。授權已確認為 MIT。
3. 實作客製環境、環境性質的測試與資料產生器，完成覆蓋率試驗（M2）。

## 未決事項

- 世界模型的起點：LeWorldModel 的提議仍待目標 GPU 上的量測。
- 各檢驗的門檻值：程序已訂定，數值在試驗量測基準線後鎖定。
- 環境參數的最終值：在覆蓋率試驗後鎖定。
- 規則搜尋方法：在上限對照中比較候選方法後選定。
- 資料集與模型檢查點的典藏服務與資料授權：第一次釋出時決定。

## 引用

引用資訊在 [CITATION.cff](CITATION.cff)。

## 參與

參與方式與規定在 [CONTRIBUTING.md](CONTRIBUTING.md)。程式碼以 [MIT 授權](LICENSE)釋出。

## 參考文獻

本文件提到的研究如下；完整的相關研究與參考文獻在 [docs/related-work.md](docs/related-work.md)。

- Held, R. and Hein, A. (1963). Movement-produced stimulation in the development of visually guided behavior. Journal of Comparative and Physiological Psychology, 56(5).
- Fedorenko, E., Piantadosi, S. T. and Gibson, E. (2024). Language is primarily a tool for communication rather than thought. Nature.
- Assran, M. et al. (2025). V-JEPA 2: Self-Supervised Video Models Enable Understanding, Prediction and Planning. arXiv:2506.09985.
- Maes, L. et al. (2026). LeWorldModel: Stable End-to-End Joint-Embedding Predictive Architecture from Pixels. arXiv:2603.19312. 官方程式碼：https://github.com/lucas-maes/le-wm
- Hao, S. et al. (2024). Training Large Language Models to Reason in a Continuous Latent Space. arXiv:2412.06769.
- Chen, D. et al. (2025). VL-JEPA: Joint Embedding Predictive Architecture for Vision-language. arXiv:2512.10942.
- Khan et al. (2026). One Life to Learn: Inferring Symbolic World Models for Stochastic Environments from Unguided Exploration. ICLR. arXiv:2510.12088.
- Courtis, D., Li, W. and Sanner, S. (2026). OPINE-World: Programmatic World Modeling with Ontology-error-Prioritized Interactive Exploration. arXiv:2607.01531. 未經同儕審查的預印本。
- Seitzer, M., Schölkopf, B. and Martius, G. (2021). Causal Influence Detection for Improving Efficiency in Reinforcement Learning. NeurIPS.
- Zhang, Y.-G., Du, T., Zhang, Q. and Wang, Y. (2026). DWM: Separating World Effects from Actions in Latent World Models. arXiv:2607.18715. 未經同儕審查的預印本。
- Farama Foundation. Minigrid. https://minigrid.farama.org/
