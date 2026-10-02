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

目前仍未被接上的缺口是：連續的表示空間與精確的規則之間，沒有不經過語言的橋。現有的串接方式，幾乎都是讓大型語言模型在中間用文字轉譯。

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

設計細節在 [design/attribution.md](docs/design/attribution.md)，決策理由在 [decisions/0004](docs/decisions/0004-rule-simulation-as-second-attribution-estimate.md)。

## 階段

依序執行，每一階段都有通過條件，未通過不進入下一階段。

| 階段 | 內容 | 回答的問題 |
| --- | --- | --- |
| 0 | 重現世界模型的公開結果 | 工具是否可用 |
| 1 | 訓練世界模型，檢查表示空間是否記錄了真實狀態 | 概念是否形成 |
| 2 | 檢查每種動作在表示空間中造成的變化 | 動作是否對應一致的轉換 |
| 3 | 從表示空間抽出規則，並以紀錄回放驗證 | 能否抽出精確規則 |
| 4 | 推廣測試、規則開關的成對比較、三方歸因比對 | 規則是否推廣得更好，歸因是否一致 |

各階段的方法與通過條件在 [design/evaluation.md](docs/design/evaluation.md)。

## 狀態

沒有任何元件已實作。設計記錄在 [docs/](docs/README.md)，尚未依據它撰寫任何程式碼。

## 下一步

1. 決定世界模型是否以 LeWorldModel 為起點。提議與理由在 [decisions/0003](docs/decisions/0003-lewm-as-world-model-starting-point.md)，目前狀態為提議中。
2. 確認 LeWorldModel 官方程式碼的授權條款，以及它的顯示記憶體用量能否符合目標硬體。
3. 重現 LeWorldModel 的其中一個公開環境，完成第零階段。
4. 細化客製環境中隱藏變數、條件式規則與規則開關的具體形式，再撰寫環境程式碼。

## 未決事項

- 神經網路歸因估計的差異量如何衡量。若表示是單一向量而非機率分布，KL 散度無法直接使用。
- 隱藏變數、條件式規則、看不見的時間規則各自的具體設計。
- 規則抽取的搜尋方法。
- 各階段通過條件的具體門檻。門檻在基準線量測完成後才設定。
- 環境紀錄的儲存格式。

## 參考文獻

- Held, R. and Hein, A. (1963). Movement-produced stimulation in the development of visually guided behavior.
- Fedorenko, E., Piantadosi, S. T. and Gibson, E. (2024). Language is primarily a tool for communication rather than thought. Nature.
- Assran, M. et al. (2025). V-JEPA 2: Self-Supervised Video Models Enable Understanding, Prediction and Planning. arXiv:2506.09985.
- Balestriero, R. and LeCun, Y. (2025). LeJEPA: Provable and Scalable Self-Supervised Learning Without the Heuristics. arXiv:2511.08544.
- Maes, L. et al. (2026). LeWorldModel: Stable End-to-End Joint-Embedding Predictive Architecture from Pixels. arXiv:2603.19312. 官方程式碼：https://github.com/lucas-maes/le-wm
- Chen, D. et al. (2025). VL-JEPA: Joint Embedding Predictive Architecture for Vision-language. arXiv:2512.10942.
- Huang, H., LeCun, Y. and Balestriero, R. (2025). LLM-JEPA: Large Language Models Meet Joint Embedding Predictive Architectures. arXiv:2509.14252.
- Hao, S. et al. (2024). Training Large Language Models to Reason in a Continuous Latent Space. arXiv:2412.06769.
- Hafner, D., Yan, W. and Lillicrap, T. (2025). Training Agents Inside of Scalable World Models. arXiv:2509.24527.
- Khan et al. (2026). One Life to Learn: Inferring Symbolic World Models for Stochastic Environments from Unguided Exploration. ICLR. arXiv:2510.12088.
- Courtis, D., Li, W. and Sanner, S. (2026). OPINE-World: Programmatic World Modeling with Ontology-error-Prioritized Interactive Exploration. arXiv:2607.01531. 未經同儕審查的預印本。
- Farama Foundation. Minigrid. https://minigrid.farama.org/
