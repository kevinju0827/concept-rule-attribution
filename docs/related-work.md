# 相關研究

本專案與哪些研究線相關、借用了什麼、差異在哪裡。根目錄的 [README](../README.md) 只列出背景；本文件是較完整的定位。

每個里程碑結束時檢查一次新發表的研究，更新本文件。最後檢查：2026-10-20。

書目資訊以一手來源為準：論文本身、出版者的頁面，或作者的官方程式碼庫。只讀過摘要、尚未讀過全文的文獻，在內文中標註；部落格、新聞稿與第三方的測試報告不作為引用來源。

## 表示空間中的世界模型

只在表示空間預測、不重建畫面的路線，起點是 LeCun（2022）提出的聯合嵌入預測架構，之後也被用於語言模型的訓練目標（Huang、LeCun 與 Balestriero，2025）。V-JEPA 2（Assran 等，2025）在影片上大規模預訓練後用於機器人規劃；LeJEPA（Balestriero 與 LeCun，2025）以一個把表示推向等向高斯分布的約束取代各種防崩塌技巧；LeWorldModel（Maes 等，2026）把這個約束用於從像素端到端訓練的世界模型，以約 1500 萬參數在單張 GPU 上訓練。

對照的路線是 Dreamer 系列（Hafner 等，2025；Hafner、Yan 與 Lillicrap，2025），它的表示是機率分布，並同時重建畫面。

C-SWM（Kipf、van der Pol 與 Welling，2020）以對比學習訓練物件分解的表示空間世界模型，並在格子世界中以「預測的表示在參考集合中排第幾」評估預測品質，不需要還原畫面。

**借用**：防崩塌的約束、只在表示空間預測、以排名評估預測品質。
**差異**：這些研究關心規劃與控制的表現；本專案關心學出的表示能否支撐結構正確、可組合的規則與自身影響的估計，並以環境的標準答案逐項檢驗。

## 在世界模型中分離環境效果與動作效果

這是與本專案的神經網路歸因最接近的研究。

DWM（Zhang、Du、Zhang 與 Wang，2026，未經同儕審查的預印本；以下依摘要整理，全文尚未查閱）把「世界效果」定義為：在相同狀態與歷史下，若把當前動作換成空動作仍會發生的變化；其餘歸為動作效果。它在表示空間世界模型上加一個學習世界效果的輸出，以正交約束與原本的預測耦合，並在 PushT、Reacher、TwoRoom 加入環境自身動態的變體上提升規劃成功率。

CAI（Seitzer、Schölkopf 與 Martius，2021）以「給定狀態下，動作與下一步狀態之間的條件互資訊」衡量代理在當下對環境的因果影響，用於改善探索，並以模擬器提供的標記檢驗偵測能力。

**借用**：以空動作為對照的分解（DWM）；以學出的模型偵測當下影響的想法（CAI）。
**差異**：

- DWM 把分解放進訓練目標以改善規劃；本專案不把歸因放進訓練目標，而是事後以複製環境得到的反事實標準答案逐步驗證，並區分無效動作、隱藏效果、預防事件與延遲效果。以 DWM 的方式訓練的模型，可以作為探索性的比較對象。
- CAI 衡量的是動作與下一步的相依程度，相當於以「所有動作的平均」為對照，結果隨代理平常的行為而變；本專案以「什麼都不做」為對照，回答的是不同的問題，理由見 [decisions/0006](decisions/0006-noop-semantics-and-contrastive-attribution.md)。動作平均對照列為探索性的基準。
- 兩者都沒有可讀的第二個估計；本專案以抽出的規則模擬，得到可以逐項檢查的歸因。

## 能動感與自身影響

神經科學中，再傳入原理（von Holst 與 Mittelstaedt，1950）與比較器模型（Blakemore、Wolpert 與 Frith，1998；Frith、Blakemore 與 Wolpert，2000）主張：大腦以動作指令的副本預測自己造成的感覺，再與實際感覺比較，以區分自己造成的變化與外界造成的變化。Haggard（2017）整理了能動感的研究。

機器學習中，相關的概念多半作為訓練或探索的訊號：辨識畫面中受代理控制的區域（Choi 等，2019）、學習可以被獨立控制的因素（Thomas 等，2017）、以動作與未來狀態之間的通道容量衡量控制力（Klyubin、Polani 與 Nehaniv，2005）。

**借用**：「預測自己動作的後果，再與另一個參考比較」的結構。
**差異**：本專案的參考是「什麼都不做」時的預測，而不是實際的感覺；並且刻意不把歸因作為訓練訊號，以免代理被引導去盡量影響世界。

## 精確性、組合性與生物的近似模型

生物的預測並不精確。人對物理場景的直覺判斷，可以由帶雜訊、近似的模擬解釋（Battaglia、Hamrick 與 Tenenbaum，2013）；人的認知可以理解為在有限的計算資源下取捨準確度（Lieder 與 Griffiths，2020）。另一方面，嬰兒對物體的預期是類別式、結構正確的，例如物體恆存與不可穿透（Spelke 與 Kinzler，2007）；而精確的數量概念要靠數詞這類符號工具（Frank、Everett、Fedorenko 與 Gibson，2008）。Lake 等（2017）主張，像人一樣學習的機器需要因果模型與組合性，而不只是模式辨識。

以獨立規則描述世界的困難早已被指出：要寫明什麼不會改變的框架問題、間接效果的衍生問題、條件永遠列不完的限定問題（McCarthy 與 Hayes，1969）。因果表示學習以「獨立的因果機制」與「分布改變時只有少數機制改變」作為可組合的基礎（Schölkopf 等，2021）；物件導向世界模型把組合推廣形式化，並以只看過部分物體組合的訓練、測試新組合（Zhao、Kong、Walters 與 Wong，2022）。

以描述長度選擇模型，是在準確度與簡潔之間取捨的標準方法（Rissanen，1978；Grünwald，2007）。

**借用**：區分數值的精確與結構的正確；以描述長度取捨保真度與複雜度；以「只看過部分機制、測試新組合」檢驗可組合性。
**差異**：本專案的機制組合在環境層級定義，每個機制可以整組關閉而不改變其餘規則，因此能乾淨地分開「規則本身」與「規則如何疊加」。

## 表示中的世界狀態：讀取與介入

在 Othello 棋局上訓練的序列模型中，讀取器能讀出棋盤狀態，沿讀取器的方向修改內部表示會改變模型的預測（Li 等，2023）；之後發現，以「目前玩家的棋子與對手的棋子」為目標時，棋盤是線性表示的，以絕對顏色為目標則否（Nanda、Lee 與 Wattenberg，2023）。Zhang（2026，研討會論文，依摘要整理）以類似方法讀取強化學習中學出的環境模擬器，發現物體位置與分數等變數大致可以線性讀出。

讀取方法本身的問題：Hewitt 與 Liang（2019）提出以對照任務衡量讀取器的選擇性；Voita 與 Titov（2020）以最小描述長度衡量讀取；Belinkov（2022）整理了讀取的限制；Elazar 等（2021）以移除資訊後的行為變化檢驗表示是否被使用。

**借用**：線性讀取加上介入檢查；描述長度；以代理為參考點的讀取目標。
**差異**：本專案的隱藏變數有標準答案與可識別性標記，並以「單張畫面上限」區分「模型整合了歷史」與「畫面中的相關線索」。對照任務不直接適用於逐格的讀取器，理由見 [decisions/0009](decisions/0009-probe-targets-and-controls.md)。

## 動作作為表示空間中的轉換

Higgins 等（2018）以對稱群定義解耦的表示；Quessard、Barrett 與 Clements（2020）學習讓動作在表示空間中以群作用表現的表示；van der Pol 等（2020）以馬可夫決策過程的同態，讓動作在表示空間中成為等變的轉換。

**與第二階段的關係**：這些研究把「動作是表示空間中的一致轉換」當作訓練目標。本專案不加入這類約束，而是檢驗一般的預測目標是否自然產生這種結構。

## 不經語言模型的符號規則學習

- 物件導向馬可夫決策過程與 DOORMAX（Diuk、Cohen 與 Littman，2008）在確定性的物件環境中學習「條件 → 效果」，並證明樣本複雜度的上界。這是本專案規則搜尋的候選方法之一。
- Pasula、Zettlemoyer 與 Kaelbling（2007）學習帶雜訊的關係規則，適用於日後的隨機變體。
- Schema Networks（Kansky 等，2017）以實體與局部規則建構可推廣的生成式模型。
- Apperception Engine（Evans 等，2021）從感覺序列建構符號化的因果理論，並能發明看不見的命題。
- AutumnSynth（Das 等，2023）從格子世界的觀測合成反應式程式，以自動機合成補上看不見的狀態。它與本專案「找回看不見的時間規則」的問題最接近。
- 以解釋轉移學習邏輯程式（Inoue、Ribeiro 與 Sakama，2014）；從失敗中學習程式的 Popper（Cropper 與 Morel，2021）。
- 以理論為基礎的強化學習（Tsividis 等，2021）以類似遊戲描述語言的模型達到人類水準的學習效率。
- CGWM（Andreoni、Ronzani、Ghidini、Daniele 與 van Krieken，2026，未經同儕審查的預印本，讀過全文）從畫面學習編碼器，把畫面對應到人預先命名的概念，再以人提供的 LTLf 規則預測轉移與規劃，不使用語言模型。它的比較對象是 LeWorldModel 加上以標準答案概念監督的線性讀取器：讀取器在單張畫面上辨識概念與 CGWM 相當，但沿著想像的軌跡往後推時 F1 下降。它把「從資料學出缺少的規則」列為未來工作，而那正是本專案要做的事。

**借用**：規則形式、搜尋方法、潛在狀態的處理方式。
**差異**：除了 CGWM，這些方法的輸入是符號：物件清單或格子內容。CGWM 的輸入是畫面，但規則由人提供。本專案的輸入是從畫面學出的表示，經讀取器轉成狀態，並量測這一步造成的損失；上限對照正是把這些方法直接用在真實狀態上的結果。

## 以語言模型建立程式形式的世界模型

WorldCoder（Tang、Key 與 Ellis，2024）、OneLife（Khan 等，2026）、OPINE-World（Courtis、Li 與 Sanner，2026，未經同儕審查的預印本）讓大型語言模型撰寫或修改描述環境的程式。WALL-E 2.0（Zhou 等，2025，未經同儕審查的預印本）讓語言模型比較預測與實際的軌跡，歸納出規則來修正語言模型的世界模型；NeSyS（Zhao 等，2026）以語言模型撰寫的規則重新排序語言模型世界模型的候選預測。

**差異**：在這些方法中，連續的觀測與精確的規則之間由語言模型以文字轉譯。本專案要檢驗的，正是不經過語言的橋是否存在，因此刻意不使用語言模型。

## 校準與選擇性預測

現代神經網路的機率常常過度自信（Guo、Pleiss、Sun 與 Weinberger，2017）；輸入分布偏移時，校準通常隨之變差（Ovadia 等，2019）。期望校準誤差以分箱計算（Naeini、Cooper 與 Hauskrecht，2015），但分箱估計會低估真正的校準誤差（Kumar、Liang 與 Ma，2019）。負對數似然與 Brier 分數（Brier，1950）是嚴格適當的評分規則（Gneiting 與 Raftery，2007），不需要分箱。允許模型在信心不足時不作答，並以風險–涵蓋率曲線評估，是選擇性預測的做法（Geifman 與 El-Yaniv，2017）。

**借用**：以適當評分規則為主、分箱指標為輔的校準量測；在分布偏移下量測校準；選擇性預測的評估方式。
**差異**：一般的校準研究只能從結果反推機率是否可信；本專案的帶雜訊變體知道每個結果的真實機率，可以直接比較。

## 實際因果

Halpern 與 Pearl（2005）、Halpern（2016）以結構因果模型定義「某事是某結果的原因」；Pearl（2009）是反事實推論的標準參考。

本專案的歸因採用最簡單的「若非如此」定義，相對於什麼都不做。它的已知盲點，例如多個充分原因同時存在時代理的貢獻被算成零，在這些文獻中有詳細討論；本專案照實回報，不另行修正。

## 慢想：直覺提出候選，深思評估

本專案的工作假設是：人在深思時，常在直覺給出的少數候選中，以學到的知識評估後再選擇。以下整理 2026-10-20 查證的證據，依假設的四個部分排列：直覺提出候選、深思評估候選、評估依據學到的規則、結果勝過在候選中隨機挑選。

**直覺提出候選。** 預設介入模型認為直覺產生一個預設反應，深思有時介入並覆寫它（Evans 與 Stanovich，2013，只讀過摘要）。三階段模型則認為一個刺激可能同時引出多個直覺反應，偵測到衝突時才啟動深思（Pennycook、Fugelsang 與 Koehler，2015，只讀過摘要）。Morris、Phillips、Huang 與 Cushman（2021，讀過預印本全文）在九個實驗中發現，受試者先以與情境無關的快取價值想出少數候選，再以情境特定的價值在其中選擇。另一方面，消防指揮官的決策點中，同時比較兩個以上選項的不到 12%（Klein、Calderwood 與 Clinton-Cirocco，1986，只讀過摘要），多數時候只想到一個選項。

**深思評估候選。** 下棋時，專家與較弱的棋手經過深思後選的棋步，都比一開始偏好的棋步客觀上更好（Moxley、Ericsson、Charness 與 Krampe，2012，只讀過摘要）。但深思真正改變答案的情況很少：先在時限與記憶負荷下作答、再讓受試者重想的實驗中，衝突題的答案改變的比例合計不到約 11%，最後答對的試次中，第一次就已答對的佔 75% 到 88%（Bago 與 De Neys，2017，讀過全文）。初始答案的正確感越強，重想越短、越少改答案（Thompson、Prowse Turner 與 Pennycook，2011，只讀過摘要）。深思也常只是替直覺找理由（Pennycook 等，2015；De Neys，2023，讀過接受稿全文）。人評估自己的論證時特別寬鬆：自己的論證被偽裝成別人的論證時，56% 到 58% 的受試者否決了它（Trouche、Johansson、Hall 與 Mercier，2016，只讀過摘要）。

**評估依據學到的規則。** 沒有研究直接比較評估者是規則還是模擬。齒輪問題中，受試者先在腦中模擬齒輪轉動，從模擬結果歸納出規則，之後直接使用規則，規則不適用時再退回模擬（Schwartz 與 Black，1996，讀過全文）。Kruglanski 與 Gigerenzer（2011，只讀過摘要）主張直覺與深思都以規則為基礎，準確與否取決於規則與環境是否相配；也有研究質疑快想與慢想的二分本身（Melnikoff 與 Bargh，2018，只讀過摘要）。直覺是否可靠，取決於環境是否規律，以及是否有機會學到其中的規律（Kahneman 與 Klein，2009，只讀過摘要）。

**勝過在候選中隨機挑選。** 查到的人類研究中，沒有一項直接比較「深思後在自己的候選中挑選」與「在自己的候選中隨機挑選」。間接證據互相矛盾：重想時錯改對的比例高於對改錯（Bago 與 De Neys，2017）；但在數量估計中，受試者在自己的兩次估計與其平均之間挑選時，只依數值線索的挑選準確度與機率相當（Fraundorf 與 Benjamin，2014，只讀過摘要）。

**借用**：以「在候選中隨機挑選」作為評估的對照；把效果集中在直覺出錯的少數情況這件事納入分析；評估者可能是模擬，規則是模擬的壓縮形式。
**差異**：本專案沒有人類受試者，不檢驗假設中人類的部分，只檢驗 AI 一側：以自己歸納出的規則評估自己的猜測，是否勝過隨機挑選與多數決。

## 以獨立的檢驗挑選候選

「先產生候選，再以另一套機制評估」在 AI 中很常見，差別在評估者從哪裡來。

- **人寫的或環境本身的檢驗。** AlphaGeometry（Trinh、Wu、Le、He 與 Luong，2024，讀過全文）由語言模型提出輔助作圖，交給以人設計的演繹規則推導的符號引擎驗證。LLM-Modulo（Kambhampati 等，2024，讀過全文）主張語言模型產生候選，由外部、可靠的檢驗者評估。以重複抽樣加上自動檢驗時，涵蓋率隨樣本數持續上升；只靠多數決或打分數模型時，在約 100 個樣本後停滯（Brown 等，2024，讀過全文）。
- **學到的神經網路檢驗。** 訓練一個檢驗模型為候選排序（Cobbe 等，2021，讀過全文）；逐步打分的模型勝過整題打分與多數決（Lightman 等，2023，讀過全文）；MuZero（Schrittwieser 等，2020，讀過全文）以學到的模型在樹搜尋中評估策略提出的動作。
- **沒有獨立檢驗時。** 對多條推理路徑取多數決可以提升表現（Wang 等，2023，讀過全文）；但沒有外部回饋時，模型自我修正無效，甚至變差（Huang 等，2024，讀過全文）。
- **自己歸納的規則。** Leiva 等（2025，只讀過摘要與部分段落）在測試時以學到的錯誤偵測規則，對多個預訓練分類器的輸出做溯因推理，勝過多數決；它處理的是分類，衝突約束由人給定。WALL-E 2.0（Zhou 等，2025）與 NeSyS（Zhao 等，2026）以語言模型歸納規則，用來修正語言模型世界模型的預測；狀態是文字，不是畫面。

**借用**：比較對象至少包括隨機挑選、多數決與學到的檢驗者。
**差異**：查到的研究都缺少下列至少一項：輸入是畫面、規則由系統自己從感知表示中歸納、不使用語言模型或人寫的規則、規則用於檢驗神經世界模型的下一步預測、並與多數決直接比較。本專案的讀取器以標準答案訓練，概念詞彙仍是人定的，只有規則是自己歸納的。

## 實驗方法

- 種子數與區間估計：Henderson 等（2018）、Colas、Sigaud 與 Oudeyer（2018）、Agarwal 等（2021）。
- 可重現性清單：Pineau 等（2021）。
- 事先登記：Nosek 等（2018）。
- 資料說明書與模型卡：Gebru 等（2021）、Mitchell 等（2019）。
- 有效秩：Roy 與 Vetterli（2007）。

## 環境

MiniGrid（Chevalier-Boisvert 等，2023）；後續檢驗預定使用的 Crafter（Hafner，2022）。

## 文件與專案結構的參考

- 決策紀錄的格式：Nygard（2011）。
- 文件依讀者的需求分類：Diátaxis。
- 程式碼的目錄與工具：Python Packaging User Guide 的 `src` 版面、Scientific Python Development Guide、The Good Research Code Handbook。
- 引用資訊：Citation File Format。

## 參考文獻

- Agarwal, R., Schwarzer, M., Castro, P. S., Courville, A. and Bellemare, M. G. (2021). Deep Reinforcement Learning at the Edge of the Statistical Precipice. NeurIPS.
- Andreoni, R., Ronzani, M., Ghidini, C., Daniele, A. and van Krieken, E. (2026). Neurosymbolic Learning of Interpretable World Models. arXiv:2610.19955. 未經同儕審查的預印本。
- Assran, M. et al. (2025). V-JEPA 2: Self-Supervised Video Models Enable Understanding, Prediction and Planning. arXiv:2506.09985.
- Bago, B. and De Neys, W. (2017). Fast logic?: Examining the time course assumption of dual process theory. Cognition, 158.
- Balestriero, R. and LeCun, Y. (2025). LeJEPA: Provable and Scalable Self-Supervised Learning Without the Heuristics. arXiv:2511.08544.
- Battaglia, P. W., Hamrick, J. B. and Tenenbaum, J. B. (2013). Simulation as an engine of physical scene understanding. PNAS, 110(45).
- Belinkov, Y. (2022). Probing Classifiers: Promises, Shortcomings, and Advances. Computational Linguistics, 48(1).
- Blakemore, S.-J., Wolpert, D. M. and Frith, C. D. (1998). Central cancellation of self-produced tickle sensation. Nature Neuroscience, 1(7).
- Brier, G. W. (1950). Verification of forecasts expressed in terms of probability. Monthly Weather Review, 78(1).
- Brown, B., Juravsky, J., Ehrlich, R., Clark, R., Le, Q. V., Ré, C. and Mirhoseini, A. (2024). Large Language Monkeys: Scaling Inference Compute with Repeated Sampling. arXiv:2407.21787.
- Chevalier-Boisvert, M. et al. (2023). Minigrid & Miniworld: Modular & Customizable Reinforcement Learning Environments for Goal-Oriented Tasks. NeurIPS Datasets and Benchmarks Track.
- Choi, J. et al. (2019). Contingency-Aware Exploration in Reinforcement Learning. ICLR.
- Cobbe, K. et al. (2021). Training Verifiers to Solve Math Word Problems. arXiv:2110.14168.
- Colas, C., Sigaud, O. and Oudeyer, P.-Y. (2018). How Many Random Seeds? Statistical Power Analysis in Deep Reinforcement Learning Experiments. arXiv:1806.08295.
- Courtis, D., Li, W. and Sanner, S. (2026). OPINE-World: Programmatic World Modeling with Ontology-error-Prioritized Interactive Exploration for ARC-AGI-3. arXiv:2607.01531. 未經同儕審查的預印本。
- Cropper, A. and Morel, R. (2021). Learning programs by learning from failures. Machine Learning, 110.
- Das, R., Tenenbaum, J. B., Solar-Lezama, A. and Tavares, Z. (2023). Combining Functional and Automata Synthesis to Discover Causal Reactive Programs. Proceedings of the ACM on Programming Languages, 7(POPL).
- De Neys, W. (2023). Advancing theorizing about fast-and-slow thinking. Behavioral and Brain Sciences, 46, e111.
- Diuk, C., Cohen, A. and Littman, M. L. (2008). An Object-Oriented Representation for Efficient Reinforcement Learning. ICML.
- Elazar, Y., Ravfogel, S., Jacovi, A. and Goldberg, Y. (2021). Amnesic Probing: Behavioral Explanation with Amnesic Counterfactuals. Transactions of the ACL, 9.
- Evans, J. St. B. T. and Stanovich, K. E. (2013). Dual-process theories of higher cognition: Advancing the debate. Perspectives on Psychological Science, 8(3).
- Evans, R., Hernández-Orallo, J., Welbl, J., Kohli, P. and Sergot, M. (2021). Making sense of sensory input. Artificial Intelligence, 293.
- Frank, M. C., Everett, D. L., Fedorenko, E. and Gibson, E. (2008). Number as a cognitive technology: Evidence from Pirahã language and cognition. Cognition, 108(3).
- Fraundorf, S. H. and Benjamin, A. S. (2014). Knowing the crowd within: Metacognitive limits on combining multiple judgments. Journal of Memory and Language, 71.
- Frith, C. D., Blakemore, S.-J. and Wolpert, D. M. (2000). Abnormalities in the awareness and control of action. Philosophical Transactions of the Royal Society B, 355.
- Gebru, T. et al. (2021). Datasheets for Datasets. Communications of the ACM, 64(12).
- Geifman, Y. and El-Yaniv, R. (2017). Selective Classification for Deep Neural Networks. NeurIPS.
- Gneiting, T. and Raftery, A. E. (2007). Strictly Proper Scoring Rules, Prediction, and Estimation. Journal of the American Statistical Association, 102(477).
- Grünwald, P. D. (2007). The Minimum Description Length Principle. MIT Press.
- Guo, C., Pleiss, G., Sun, Y. and Weinberger, K. Q. (2017). On Calibration of Modern Neural Networks. ICML.
- Hafner, D. (2022). Benchmarking the Spectrum of Agent Capabilities. ICLR.
- Hafner, D., Pasukonis, J., Ba, J. and Lillicrap, T. (2025). Mastering diverse control tasks through world models. Nature, 640.
- Hafner, D., Yan, W. and Lillicrap, T. (2025). Training Agents Inside of Scalable World Models. arXiv:2509.24527.
- Haggard, P. (2017). Sense of agency in the human brain. Nature Reviews Neuroscience, 18.
- Halpern, J. Y. (2016). Actual Causality. MIT Press.
- Halpern, J. Y. and Pearl, J. (2005). Causes and Explanations: A Structural-Model Approach. Part I: Causes. The British Journal for the Philosophy of Science, 56(4).
- Henderson, P., Islam, R., Bachman, P., Pineau, J., Precup, D. and Meger, D. (2018). Deep Reinforcement Learning that Matters. AAAI.
- Hewitt, J. and Liang, P. (2019). Designing and Interpreting Probes with Control Tasks. EMNLP-IJCNLP.
- Higgins, I. et al. (2018). Towards a Definition of Disentangled Representations. arXiv:1812.02230.
- Huang, H., LeCun, Y. and Balestriero, R. (2025). LLM-JEPA: Large Language Models Meet Joint Embedding Predictive Architectures. arXiv:2509.14252.
- Huang, J., Chen, X., Mishra, S., Zheng, H. S., Yu, A. W., Song, X. and Zhou, D. (2024). Large Language Models Cannot Self-Correct Reasoning Yet. ICLR. arXiv:2310.01798.
- Inoue, K., Ribeiro, T. and Sakama, C. (2014). Learning from interpretation transition. Machine Learning, 94.
- Kahneman, D. and Klein, G. (2009). Conditions for intuitive expertise: A failure to disagree. American Psychologist, 64(6).
- Kambhampati, S. et al. (2024). LLMs Can't Plan, But Can Help Planning in LLM-Modulo Frameworks. ICML. arXiv:2402.01817.
- Kansky, K. et al. (2017). Schema Networks: Zero-shot Transfer with a Generative Causal Model of Intuitive Physics. ICML.
- Khan et al. (2026). One Life to Learn: Inferring Symbolic World Models for Stochastic Environments from Unguided Exploration. ICLR. arXiv:2510.12088.
- Kipf, T., van der Pol, E. and Welling, M. (2020). Contrastive Learning of Structured World Models. ICLR.
- Klein, G. A., Calderwood, R. and Clinton-Cirocco, A. (1986). Rapid Decision Making on the Fire Ground. Proceedings of the Human Factors Society Annual Meeting, 30.
- Klyubin, A. S., Polani, D. and Nehaniv, C. L. (2005). Empowerment: A Universal Agent-Centric Measure of Control. IEEE Congress on Evolutionary Computation.
- Kruglanski, A. W. and Gigerenzer, G. (2011). Intuitive and deliberate judgments are based on common principles. Psychological Review, 118(1).
- Kumar, A., Liang, P. and Ma, T. (2019). Verified Uncertainty Calibration. NeurIPS.
- Lake, B. M., Ullman, T. D., Tenenbaum, J. B. and Gershman, S. J. (2017). Building machines that learn and think like people. Behavioral and Brain Sciences, 40.
- LeCun, Y. (2022). A Path Towards Autonomous Machine Intelligence. OpenReview.
- Leiva, M. et al. (2025). Consistency-based Abductive Reasoning over Perceptual Errors of Multiple Pre-trained Models in Novel Environments. arXiv:2505.19361.
- Li, K., Hopkins, A. K., Bau, D., Viégas, F., Pfister, H. and Wattenberg, M. (2023). Emergent World Representations: Exploring a Sequence Model Trained on a Synthetic Task. ICLR.
- Lieder, F. and Griffiths, T. L. (2020). Resource-rational analysis: Understanding human cognition as the optimal use of limited computational resources. Behavioral and Brain Sciences, 43.
- Lightman, H. et al. (2023). Let's Verify Step by Step. arXiv:2305.20050.
- Maes, L. et al. (2026). LeWorldModel: Stable End-to-End Joint-Embedding Predictive Architecture from Pixels. arXiv:2603.19312.
- McCarthy, J. and Hayes, P. J. (1969). Some philosophical problems from the standpoint of artificial intelligence. Machine Intelligence, 4.
- Melnikoff, D. E. and Bargh, J. A. (2018). The Mythical Number Two. Trends in Cognitive Sciences, 22(4).
- Mitchell, M. et al. (2019). Model Cards for Model Reporting. FAT*.
- Morris, A., Phillips, J., Huang, K. and Cushman, F. (2021). Generating Options and Choosing Between Them Depend on Distinct Forms of Value Representation. Psychological Science, 32(11).
- Moxley, J. H., Ericsson, K. A., Charness, N. and Krampe, R. T. (2012). The role of intuition and deliberative thinking in experts' superior tactical decision-making. Cognition, 124(1).
- Naeini, M. P., Cooper, G. F. and Hauskrecht, M. (2015). Obtaining Well Calibrated Probabilities Using Bayesian Binning. AAAI.
- Nanda, N., Lee, A. and Wattenberg, M. (2023). Emergent Linear Representations in World Models of Self-Supervised Sequence Models. BlackboxNLP.
- Nosek, B. A., Ebersole, C. R., DeHaven, A. C. and Mellor, D. T. (2018). The preregistration revolution. PNAS, 115(11).
- Nygard, M. (2011). Documenting Architecture Decisions.
- Ovadia, Y. et al. (2019). Can You Trust Your Model's Uncertainty? Evaluating Predictive Uncertainty Under Dataset Shift. NeurIPS.
- Pasula, H. M., Zettlemoyer, L. S. and Kaelbling, L. P. (2007). Learning Symbolic Models of Stochastic Domains. Journal of Artificial Intelligence Research, 29.
- Pearl, J. (2009). Causality: Models, Reasoning, and Inference (2nd ed.). Cambridge University Press.
- Pennycook, G., Fugelsang, J. A. and Koehler, D. J. (2015). What makes us think? A three-stage dual-process model of analytic engagement. Cognitive Psychology, 80.
- Pineau, J. et al. (2021). Improving Reproducibility in Machine Learning Research (A Report from the NeurIPS 2019 Reproducibility Program). JMLR, 22.
- Quessard, R., Barrett, T. D. and Clements, W. R. (2020). Learning Disentangled Representations and Group Structure of Dynamical Environments. NeurIPS.
- Rissanen, J. (1978). Modeling by shortest data description. Automatica, 14(5).
- Roy, O. and Vetterli, M. (2007). The effective rank: A measure of effective dimensionality. EUSIPCO.
- Schrittwieser, J. et al. (2020). Mastering Atari, Go, chess and shogi by planning with a learned model. Nature, 588.
- Schwartz, D. L. and Black, J. B. (1996). Shuttling Between Depictive Models and Abstract Rules: Induction and Fallback. Cognitive Science, 20(4).
- Schölkopf, B., Locatello, F., Bauer, S., Ke, N. R., Kalchbrenner, N., Goyal, A. and Bengio, Y. (2021). Toward Causal Representation Learning. Proceedings of the IEEE, 109(5).
- Seitzer, M., Schölkopf, B. and Martius, G. (2021). Causal Influence Detection for Improving Efficiency in Reinforcement Learning. NeurIPS.
- Spelke, E. S. and Kinzler, K. D. (2007). Core knowledge. Developmental Science, 10(1).
- Tang, H., Key, D. and Ellis, K. (2024). WorldCoder, a Model-Based LLM Agent: Building World Models by Writing Code and Interacting with the Environment. NeurIPS.
- Thomas, V. et al. (2017). Independently Controllable Factors. arXiv:1708.01289.
- Thompson, V. A., Prowse Turner, J. A. and Pennycook, G. (2011). Intuition, reason, and metacognition. Cognitive Psychology, 63(3).
- Trinh, T. H., Wu, Y., Le, Q. V., He, H. and Luong, T. (2024). Solving olympiad geometry without human demonstrations. Nature, 625.
- Trouche, E., Johansson, P., Hall, L. and Mercier, H. (2016). The Selective Laziness of Reasoning. Cognitive Science, 40(8).
- Tsividis, P. A. et al. (2021). Human-Level Reinforcement Learning through Theory-Based Modeling, Exploration, and Planning. arXiv:2107.12544.
- van der Pol, E., Kipf, T., Oliehoek, F. A. and Welling, M. (2020). Plannable Approximations to MDP Homomorphisms: Equivariance under Actions. AAMAS.
- Voita, E. and Titov, I. (2020). Information-Theoretic Probing with Minimum Description Length. EMNLP.
- von Holst, E. and Mittelstaedt, H. (1950). Das Reafferenzprinzip. Naturwissenschaften, 37.
- Wang, X. et al. (2023). Self-Consistency Improves Chain of Thought Reasoning in Language Models. ICLR. arXiv:2203.11171.
- Zhang, X. (2026). What Do World Models Learn in RL? Probing Latent Representations in Learned Environment Simulators. ICLR 2026 Workshop on World Models. arXiv:2603.21546.
- Zhang, Y.-G., Du, T., Zhang, Q. and Wang, Y. (2026). DWM: Separating World Effects from Actions in Latent World Models. arXiv:2607.18715. 未經同儕審查的預印本。
- Zhao, H., Zhou, S., Yang, H., Qin, Z. and Zhou, T. (2026). Neuro-Symbolic Synergy for World Modeling. arXiv:2602.10480.
- Zhao, L., Kong, L., Walters, R. and Wong, L. L. S. (2022). Toward Compositional Generalization in Object-Oriented World Modeling. ICML.
- Zhou, S., Zhou, T., Yang, Y., Long, G., Ye, D., Jiang, J. and Zhang, C. (2025). WALL-E 2.0: World Alignment by NeuroSymbolic Learning improves World Model-based LLM Agents. arXiv:2504.15785. 未經同儕審查的預印本。

### 實務指南

- Diátaxis. https://diataxis.fr/
- Python Packaging User Guide: src layout vs flat layout. https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/
- Scientific Python Development Guide. https://learn.scientific-python.org/development/
- The Good Research Code Handbook. https://goodresearch.dev/
- Citation File Format. https://citation-file-format.github.io/
- Keep a Changelog. https://keepachangelog.com/
- Semantic Versioning. https://semver.org/
