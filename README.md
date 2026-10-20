# concept-rule-attribution

在規則完全已知的格子世界中，讓 AI 從畫面經驗自己整理出一套世界的模擬或規則，用來判斷自己的直覺選項是否合理，並以環境的標準答案逐項檢驗。支撐這個目標的有三件事：從畫面學出的表示空間是否形成概念；能否抽出結構正確、可驗證、可組合的規則；自己整理的模擬能否正確估計環境變化中有多少是自己造成的。

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

## 工作假設

人在深思時，常在直覺給出的少數候選中，以學到的知識評估後再選擇；評估的依據可能是腦中的模擬，也可能是從模擬中歸納出的規則。這在人類身上只有部分證據，「是否勝過隨機挑選」尚未被直接檢驗。

目前的 AI 並非沒有「先產生候選、再評估」的做法，但評估者幾乎都是人寫的規則、環境本身，或另一個神經網路。本專案的目標是補上 AI 一側：讓 AI 自己從經驗中整理出一套世界的模擬或規則，以它評估自己的直覺選項，並檢驗這樣做能否勝過在候選中隨機挑選與多數決。評估者是明確的規則還是學到的神經網路模擬，不預設答案，兩者都是比較的對象。

以語言模型作類比：預測下一個字的機率是最小單位的直覺；思考模式在沒有外部資料的情況下先寫計畫再執行，計畫是一連串只靠自己產生的直覺，中間沒有獨立的檢查。本專案的世界模型預測下一步、沿著動作往後預測多步、在想像中規劃選動作，分別對應這三者，並依序檢驗。這只是類比，本專案不直接檢驗語言模型。

本專案沒有人類受試者，不檢驗假設中關於人的部分；讀取器以標準答案訓練，概念詞彙仍是人定的，只有模擬或規則由系統自己整理。

檢驗的方法尚未決定。查證的證據記錄在 [decisions/0018](docs/decisions/0018-intuition-candidates-and-rule-evaluation-hypothesis.md)，目標的調整記錄在 [decisions/0019](docs/decisions/0019-goal-self-built-evaluator-for-intuitive-candidates.md)，直覺選項的三層範圍記錄在 [decisions/0020](docs/decisions/0020-scope-of-intuitive-candidates.md)，相關研究整理在 [docs/related-work.md](docs/related-work.md)。

## 為什麼使用格子世界

本專案在一個所有變數都可控、標準答案可直接取得的格子世界中，讓 AI 自己整理世界的模擬或規則，並逐項檢驗它是否正確、能否用來判斷直覺選項、能否正確估計自己造成的變化。這些機制若在這裡不成立，更複雜的環境也不會讓它們成立；若成立，之後要回答的問題就收斂成「這些機制在真實的多來源混淆下是否仍然有效」。

合併研究規則抽取與歸因的理由記錄在 [decisions/0001](docs/decisions/0001-combine-rule-extraction-and-attribution.md)。

## 研究問題

**主問題。** AI 以自己從經驗中整理出的模擬或規則評估自己的直覺選項，能否勝過在候選中隨機挑選與多數決？明確的規則與學到的神經網路模擬，哪一種更適合當評估者？直覺選項分三層依序檢驗：一步預測、多步預測、以世界模型規劃選動作；其餘方法尚未決定。

以下三個問題支撐主問題：

1. **概念。** 不經文字、直接從畫面學出的表示空間，是否形成對應真實狀態的概念，包括看不見的隱藏變數，以及屬於代理自身與屬於其他物體的變數？
2. **規則。** 動作在表示空間中是否對應一致的轉換？能否不經大型語言模型，直接抽出結構正確、可組合的規則？抽出的規則是否比神經網路更能推廣到沒看過的情況，包括機制的新組合與帶雜訊的世界？
3. **歸因。** 自己整理的模擬能否正確回答「如果我換個做法會怎樣」：神經網路的反事實歸因估計、以規則模擬得到的歸因估計，以及環境提供的標準答案，三者是否一致？

## 「精確」指的是什麼

生物並沒有精確的預判能力：人對物理場景的直覺判斷，可以由帶雜訊的近似模擬解釋（Battaglia 等，2013）。但生物的概念在結構上往往是對的，而且是類別式的，例如物體不會憑空消失、實體不能互相穿透（Spelke 與 Kinzler，2007）。小孩不知道箱子會滑多遠，但知道後面被擋就推不動。

本專案因此區分兩種精確：數值的精確不是目標，以保真度連續量測；結構的正確才是目標，在規則已知的環境中可以逐條判定。精確的結構仍然重要：人類精確的數量概念要靠數詞這類符號工具（Frank 等，2008），這表示離散、精確的結構可能正是強邏輯能力的來源之一，而本專案要問的是，這種結構能不能不經過語言、直接從表示中得到。

以一條條規則描述世界，在複雜環境中有已知的困難：規則同時作用、互相限制，條件永遠列不完（McCarthy 與 Hayes，1969）。所以本專案不只問規則能否抽出，也問分別學到的規則在新的組合中是否仍然適用，以及在帶雜訊的世界中，「夠用」的規則是否仍然抓到正確的結構。

理由與取捨記錄在 [decisions/0014](docs/decisions/0014-structural-correctness-and-mdl-rule-selection.md) 與 [decisions/0015](docs/decisions/0015-generalization-tests-with-mechanism-composition.md)。

## 歸因與主問題的連結

判斷一個直覺選項是否合理，與估計自己造成了什麼，用的是同一種能力：以自己整理的模擬，比較不同做法的結果。歸因比較的是「實際做的動作」與「什麼都不做」，而且每一步都有標準答案，所以它是檢驗這種能力最直接的地方。

如果能抽出結構正確的規則，就能把規則分成兩類：「什麼都不做」時環境自己如何變化，以及每個動作如何改變狀態。用規則分別模擬「實際做的動作」和「什麼都不做」，兩者的差就是代理自身的影響。這和神經網路的反事實歸因回答的是同一個問題，只是以可讀、可檢查的形式回答。

規則解釋不了的變化，不能直接當作代理自己造成的。它混合了三種來源：沒抽完整的動作規則、規則沒涵蓋的其他來源，以及規則本身的誤差。區分它們的方法，是主動改變自己的動作，看剩餘的變化是否跟著改變。

設計細節在 [design/attribution.md](docs/design/attribution.md)，決策理由在 [decisions/0004](docs/decisions/0004-rule-simulation-as-second-attribution-estimate.md) 與 [decisions/0006](docs/decisions/0006-noop-semantics-and-contrastive-attribution.md)。

## 環境

以 MiniGrid 自建的 8×8 格子世界：代理可以推動箱子、擊中箱子、走上充電格；巡邏球依固定規則來回移動。箱子的耐久度、代理的能量與推動後的冷卻都不畫在畫面上，只能從歷史推得。

規則分成箱子、能量、冷卻、巡邏球四個機制，每個機制可以整體關閉而不改變其餘規則：關閉冷卻用於規則開關的成對比較；只以部分機制訓練、再在機制的新組合上測試，用於檢驗規則疊加時是否仍然適用。主要的版本是確定性的，複製環境即可得到任何動作的精確反事實結果；另有一個帶雜訊的變體，用於檢驗「夠用」的規則。

規則表與設計在 [design/environment.md](docs/design/environment.md)，取捨在 [decisions/0016](docs/decisions/0016-environment-mechanisms-and-stochastic-variant.md)。

## 階段

各項檢驗依相依關係推進，每一項都有事先鎖定的通過條件；未通過的檢驗只擋住依賴它的檢驗，且不以擴大規模繞過。

| 階段 | 內容 | 回答的問題 |
| --- | --- | --- |
| 0 | 重現世界模型的公開結果；驗收客製環境與資料 | 工具與環境是否可用 |
| 1 | 訓練世界模型，檢查表示是否記錄了真實狀態，包括隱藏變數 | 概念是否形成 |
| 歸因檢驗 | 比較神經網路的反事實歸因與標準答案；只依賴第一階段 | 自己學到的模擬能否正確回答反事實 |
| 2 | 檢查每種動作在表示空間中造成的變化 | 動作是否對應一致的轉換 |
| 3 | 從表示抽出規則，以描述長度選擇規則集，以真實狀態判定結構是否正確 | 能否抽出結構正確的規則 |
| 4 | 分級推廣、規則開關的成對比較、三方歸因比對、機制組合、帶雜訊的世界 | 規則是否推廣與組合得更好，歸因是否一致 |

主問題的檢驗方法尚未決定，決定後加入上表。各階段的方法與通過條件在 [design/evaluation.md](docs/design/evaluation.md)，統計方法與門檻鎖定程序在 [design/statistics.md](docs/design/statistics.md)。

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
- 以自己整理的模擬或規則評估直覺候選的方法：評估者的形式、保存結構、學習與使用方式、候選的來源與比較對象，以及規劃所需的目標任務與規劃器，討論後另寫決策紀錄。
- 資料集與模型檢查點的典藏服務與資料授權：第一次釋出時決定。

## 引用

引用資訊在 [CITATION.cff](CITATION.cff)。

## 參與

參與方式與規定在 [CONTRIBUTING.md](CONTRIBUTING.md)。程式碼以 [MIT 授權](LICENSE)釋出。

## 參考文獻

本文件提到的研究如下；完整的相關研究與參考文獻在 [docs/related-work.md](docs/related-work.md)。

- Held, R. and Hein, A. (1963). Movement-produced stimulation in the development of visually guided behavior. Journal of Comparative and Physiological Psychology, 56(5).
- Fedorenko, E., Piantadosi, S. T. and Gibson, E. (2024). Language is primarily a tool for communication rather than thought. Nature.
- Battaglia, P. W., Hamrick, J. B. and Tenenbaum, J. B. (2013). Simulation as an engine of physical scene understanding. PNAS, 110(45).
- Spelke, E. S. and Kinzler, K. D. (2007). Core knowledge. Developmental Science, 10(1).
- Frank, M. C., Everett, D. L., Fedorenko, E. and Gibson, E. (2008). Number as a cognitive technology: Evidence from Pirahã language and cognition. Cognition, 108(3).
- McCarthy, J. and Hayes, P. J. (1969). Some philosophical problems from the standpoint of artificial intelligence. Machine Intelligence, 4.
- Assran, M. et al. (2025). V-JEPA 2: Self-Supervised Video Models Enable Understanding, Prediction and Planning. arXiv:2506.09985.
- Maes, L. et al. (2026). LeWorldModel: Stable End-to-End Joint-Embedding Predictive Architecture from Pixels. arXiv:2603.19312. 官方程式碼：https://github.com/lucas-maes/le-wm
- Hao, S. et al. (2024). Training Large Language Models to Reason in a Continuous Latent Space. arXiv:2412.06769.
- Chen, D. et al. (2025). VL-JEPA: Joint Embedding Predictive Architecture for Vision-language. arXiv:2512.10942.
- Huang, H., LeCun, Y. and Balestriero, R. (2025). LLM-JEPA: Large Language Models Meet Joint Embedding Predictive Architectures. arXiv:2509.14252.
- Khan et al. (2026). One Life to Learn: Inferring Symbolic World Models for Stochastic Environments from Unguided Exploration. ICLR. arXiv:2510.12088.
- Courtis, D., Li, W. and Sanner, S. (2026). OPINE-World: Programmatic World Modeling with Ontology-error-Prioritized Interactive Exploration for ARC-AGI-3. arXiv:2607.01531. 未經同儕審查的預印本。
- Seitzer, M., Schölkopf, B. and Martius, G. (2021). Causal Influence Detection for Improving Efficiency in Reinforcement Learning. NeurIPS.
- Zhang, Y.-G., Du, T., Zhang, Q. and Wang, Y. (2026). DWM: Separating World Effects from Actions in Latent World Models. arXiv:2607.18715. 未經同儕審查的預印本。
- Farama Foundation. Minigrid. https://minigrid.farama.org/
