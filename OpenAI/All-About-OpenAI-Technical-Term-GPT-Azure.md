# 與 Azure OpenAI Server 相關技術名詞彙整

(2023/04/07 出版)

## 一句話總結 OpenAI 提供的服務

提供三種應用模式，透過 Prompt-Completion 模式 ，使用自然語言來輸入，而後依據不同模型目的，產生出該模型所期望的相關的內容(文字、程式碼、圖片)，這裡將會使用到相關的模組來提供這些服務，這包含了 GPT / Codex / DALL-E

### GPT-3

取得 Prompt 內容，並且生成 Completion 文字內容

### Codex

取得 Prompt 內容，並且生成 Completion 相關程式碼

### DALL-E

取得 Prompt 內容，並且生成 Completion 圖片

## # 甚麼是 生成式 AI

Generative AI: a broad description of AI that gnerates data, such as text, images, audio, video, code

"Generative AI" 是指一類 AI 技術，它能夠生成新的數據，例如文本、圖像、音頻、視頻或代碼。這些數據不是從現有數據中選擇或排列出來的，而是由 AI 系統自己創造出來的。這種技術通常基於 [機器學習] 或 [深度學習] 模型，它們使用現有的數據訓練出一個模型，然後使用該模型來生成新的數據。

此外，"Generative AI" 也包括將現有數據轉換為不同形式的技術，例如圖像風格轉換或文本翻譯。這些技術將現有的數據轉換為不同的形式，但不會生成全新的數據。

需要注意的是，由於這種技術能夠生成具有潛在偏見或有害內容的數據，因此需要謹慎使用。有些國家也對其使用進行了限制，以保障公共利益和個人隱私。

## 甚麼是 GPT

## GPT

GPT是 Generative Pre-trained Transformer 「生成式預訓練變換器模型」的簡稱，翻成白話文，就是預先訓練好，可以用來產生資料的一種 Transformer 模型 (這是基於谷歌開發的 [Transformer 語言模型](https://zh.wikipedia.org/zh-tw/Transformer%E6%A8%A1%E5%9E%8B)，Transformer模型於2017年由Google大腦的一個團隊推出，現已逐步取代長短期記憶（LSTM）等RNN模型成為了NLP問題的首選模型。)

所謂 Transformer 模型，並不是指模型本身會變化，而是指模型會對資料進行轉換，來產生所謂「關注度」，進而用關注度來產生預測的結果。

## GPT 擅長於那些方面工作

* Content Generation
* Summarization
* Classification
* Categorization
* Sentiment Analysis
* Data Extraction
* Translation

## Azure OpenAI 中目前可用的模型系列

* GPT-4

  一組模型可改善 GPT-3.5，並可瞭解並產生自然語言和程式碼。

* GPT-3	

  一系列可以了解並產生自然語言的模型。 這些模型有 [Davinci]、[Curie]、[Babbage]、[Ada]
  
* Codex	

  一系列可以了解並產生程式碼的模型，包括將自然語言轉譯成程式碼。

* 內嵌	
  
  一系列可以了解並使用內嵌的模型。 
  
  內嵌是一種特殊的資料表示格式，可供機器學習模型和演算法輕鬆使用。 內嵌是文字片段語意的資訊密集表示法。 
  
  目前我們為不同功能提供三種系列的內嵌模型： [**相似度**] 、 [**文字搜尋**] 和 [**程式碼搜尋**]。

## GPT-3 四種模型

GPT-3 模型可以了解並產生自然語言。 此服務提供四種模型功能，各具有不同層級的能力與速度，適用於不同的工作。 Davinci 是功能最佳的模型，而 Ada 則是速度最快的模型。

### Davinci
Davinci 是功能最佳的模型，其可以執行其他模型可執行的任何工作，而且使用較少的指示。 對於需要深度了解內容的應用程式，例如針對特定對象的摘要和創意內容產生，Davinci 會產生最佳結果。 Davinci 提供的增加功能需要更多計算資源，因此 Davinci 的成本更高，而且無法與其他模型一樣快。

Davinci 擅長的另一個領域是了解文字的意圖。 Davinci 擅長解決多種邏輯問題和說明人物的動機。 Davinci 已經能夠解決一些涉及因果關係的最具挑戰性 AI 問題。

適用於：複雜意圖、因果關係、對象摘要 (Complex intent, cause and effect, summarization for audience)

### Curie
Curie 不僅功能強大，也很快速。 雖然 Davinci 在分析複雜文字方面更強，但 Curie 對於許多細微的工作相當有能力，例如情感分類和摘要。 Curie 也精於回答問題和執行問與答，以及作為一般服務聊天機器人。

適用於：語言翻譯、複雜分類、文字情感、摘要 (Lauguage translation, complex classification, text sentiment summarization)

### Babbage
Babbage 可以執行簡單的工作，例如簡單的分類。 當涉及到語意搜尋排名文件與搜尋查詢的匹配程度時，其也相當有能力。

適用於：中等分類、語意搜尋分類 (Moderate classification, semantic search classification)

### Ada
Ada 通常是速度最快的模型，而且可以執行如下工作：剖析文字、位址更正，以及不需要太多細微差別的特定類型分類工作。 Ada 的效能通常可以藉由提供更多內容來改善。

適用於：剖析文字、簡單分類、位址更正、關鍵字 (Parsing text, simple classification, address correction, keyword)

## 什麼是模型

在機器學習中，模型是一個數學公式或算法，它可以從數據中學習並生成預測或結果。

## Azure OpenAI服務是什麼

Azure OpenAI 服務是一種進階語言 AI，為客戶提供具有 Azure 安全性和企業承諾的 OpenAI GPT-4、GPT-3、Codex 和 DALL-E 模型。

Azure OpenAI 服務提供 OpenAI 強大語言模型的 REST API 存取權，包括 GPT-3、Codex 和 Embeddings 模型系列。

### Azure OpenAI 服務處理哪些數據？

* 提示和完成

  提示由用戶提交，完成由服務通過完成（/completions、/chat/completions）和嵌入操作輸出。
* 培訓和驗證數據

  您可以提供自己的訓練數據，其中包含提示完成對，以便微調 OpenAI 模型。
* 來自訓練過程的結果數據

  在訓練一個微調模型後，該服務將輸出作業的元數據，其中包括每一步處理的令牌和驗證分

### 用於微調 OpenAI 模型的訓練數據

通過 Azure OpenAI Studio 提交給 Fine-tunes API 的訓練數據（提示完成對）使用自動化工具進行預處理以進行質量檢查，客戶提供的訓練數據僅用於微調客戶的模型，微軟不會使用它來訓練或改進任何微軟模型。

### 文本提示生成補全和嵌入結果

在客戶的 Azure OpenAI 資源中配置模型部署（由客戶的微調模型或基本模型端點組成）後，客戶可以通過 REST API、客戶端庫使用我們的完成或嵌入操作向模型提交文本提示，或 Azure OpenAI Studio；該模型生成通過 API 返回的文本輸出（完成）。在這些操作期間，模型中不會存儲任何提示或完成，並且不會使用提示和完成來訓練、重新訓練或改進模型。

### 您是否使用我的公司資料來定型任何模型？

Azure OpenAI 不會使用客戶資料來重新定型模型。 如需詳細資訊，請參閱 [Azure OpenAI 資料、隱私權和安全性指南](https://learn.microsoft.com/zh-tw/legal/cognitive-services/openai/data-privacy?context=%2Fazure%2Fcognitive-services%2Fopenai%2Fcontext%2Fcontext)。

### Azure OpenAI 處理的客戶數據是否發送到 OpenAI？

不會。Microsoft 在我們的 Azure 基礎架構中託管 OpenAI 模型，並且發送到 Azure OpenAI 的所有客戶數據都保留在 Azure OpenAI 服務中。

### 客戶數據是否用於訓練 OpenAI 模型？

不會。我們不會使用客戶數據來訓練、再訓練或改進 Azure OpenAI 服務中的模型。

### 防止濫用和有害內容的產生

Azure OpenAI 服務包括一個內容管理系統，它與模型一起工作以過濾潛在的有害內容。該系統的工作原理是通過一組旨在檢測誤用的分類模型來運行輸入提示和生成的完成。如果系統識別出有害內容，如果提示被認為不合適，客戶會在 API 調用中收到錯誤，或者響應中的 finish_reason 將是 content_filter 以表示某些生成已被過濾。

除了同步內容過濾之外，Azure OpenAI 服務還將服務的提示和完成情況存儲最多三十 (30) 天，以監控暗示以可能違反適用產品條款的方式使用服務的內容和/或行為。獲得授權的 Microsoft 員工可以查看觸發我們的自動化系統的提示和完成數據，以調查和驗證潛在的濫用行為。

# 甚麼是 Prompt Engineering

Prompt Engineering: Crafting ideal inputs in order to get the most out of large language models ( LLM )

您提到的論述是關於 "Prompt Engineering" ，即利用恰當的輸入來最大限度地發揮大型語言模型（LLM）的潛力。這個說法是正確的。Prompt Engineering 是一種策略，通過優化用戶提供給語言模型的輸入，以獲得更好、更相關的輸出。簡而言之，Prompt Engineering 是一種技巧，用於設計和調整輸入，以使模型產生最理想的回應。

對於 Prompt Engineering，有幾點需要補充：

* 清晰和具體

  為了獲得最佳結果，輸入應該越清晰、越具體越好。這有助於避免模型產生含糊不清或無關的回答。
* 問題類型

  了解您要解決的問題類型，這將幫助您選擇最合適的方式來引導模型。例如，您可以選擇多次迭代、對話形式或者將問題分解成更小的部分。
* 控制篇幅

  可以通過設定字數限制或其他篇幅要求，來控制模型生成的回答篇幅。這有助於確保回答的簡潔和實用性。
* 設定角色和背景

  為模型提供角色和背景資訊，可以幫助它產生更符合場景的回答。例如，您可以要求模型扮演某個專家，或者遵循特定的寫作風格。
* 迭代和嘗試

  Prompt Engineering 可能需要多次嘗試和調整，以找到最合適的輸入。通過不斷嘗試，您將更了解如何引導模型產生理想的回答。

總之，Prompt Engineering 是一個有效的方法，通過優化輸入以從大型語言模型中獲得更好的結果。了解如何設計和調整輸入將有助於您充分利用這些強大的模型。

# Token

Token: A unit easily understood by a language model

您提到的論述是關於 "token" ，它是語言模型容易理解的基本單位。這個說法是正確的。在自然語言處理（NLP）領域，token 是將文本分解成更小、可管理的單位的過程。根據不同的語言和模型，token 可能是單詞、字符、子詞或者詞素等。

對於 token，有幾點需要補充：

* 分詞（Tokenization）

  將原始文本分解為 token 的過程稱為分詞。分詞算法可以基於單詞、字符或其他語言單位來運作。分詞是 NLP 中的一個基本步驟，因為它將文本轉換為模型可以處理的格式。
* 語言模型的訓練

  在訓練語言模型時，token 被用作輸入和輸出。模型會學習 token 之間的關係，以便在給定上下文的情況下，生成或預測下一個最可能出現的 token。
* 語言和編碼

  不同語言的 token 編碼方式可能有所不同。例如，英文通常使用基於單詞的 token，而像中文這樣的語言則可能使用基於字符的 token。部分語言模型，如 GPT-3 和 BERT，使用了一種稱為 "Byte Pair Encoding" （BPE）或 "WordPiece" 的子詞分詞方法，以更好地處理不同語言和詞彙的多樣性。
* Token 的數量限制

  大多數語言模型都有一個固定的最大 token 數量限制，這意味著在單次輸入或輸出中，它們只能處理有限數量的 token。超過此限制可能會導致性能下降或無法正常運作。

總之，token 是語言模型容易理解的基本單位，對於 NLP 任務至關重要。理解 token 的概念以及如何適當地將文本分解為 token 對於有效地使用語言模型非常重要。

## 甚麼是 GPT Embedding

GPT Embedding 是指 GPT (Generative Pre-trained Transformer) 模型中使用的字詞嵌入 (word embedding) 技術。字詞嵌入是一種將自然語言中的單詞映射到低維向量空間中的技術。

GPT模型使用基於 Transformer 結構的深度神經網絡來進行語言建模，其中的字詞嵌入層用來將輸入的單詞轉換為向量表示，以便進一步進行神經網絡的計算。

GPT Embedding 是通過一個基於 Transformer 結構的編碼器，將每個單詞轉換為一個固定維度的向量表示。**這個向量表示的特點是：不同單詞之間的距離反映了它們在語言上的相似度**。在 GPT 模型中，字詞嵌入層是通過大量無監督的語料庫進行預訓練，以學習單詞之間的語言結構和關係。在實際應用中，這些預訓練的字詞嵌入可以幫助 GPT 模型更好地理解和生成自然語言文本。

## GPT 中的無監督學習與有監督學習有甚麼差別

監督式學習和非監督式學習的主要區別在於是否使用標記數據集。監督式學習使用標記數據，這意味著人類監督參與管理訓練數據。非監督式學習使用未標記數據，這意味著管理訓練數據時人類監督最少。

## 為什麼進行 Fine Tuning，不要做這樣事情 : Not bulk teaching of new knowledge

微調是一種遷移學習形式，它使用預先訓練的模型和現有數據來進一步訓練和提高模型的性能。 當可用於訓練的數據有限以及模型需要專門用於特定任務或數據集時，這是一種有用的技術。 它不適合新知識的批量教學，因為預訓練模型已經針對特定任務進行了優化。

進行 Fine Tuning 的目的是在一個已經訓練好的模型基礎上，進一步調整模型以適應新的任務或是特定的應用。在進行 Fine Tuning 時，通常會使用一個較小的數據集來重新訓練模型，而這個數據集通常包含了新的任務或是特定應用所需的特徵和知識。

相對於 "bulk teaching of new knowledge"，進行 Fine Tuning 的優勢在於它可以更好地保留模型原有的知識和能力。這是因為 Fine Tuning 的過程是在一個已經訓練好的模型基礎上進行微調，所以它可以利用原有的知識和能力來更好地適應新的任務或應用。

另一方面，"bulk teaching of new knowledge" 則是直接使用大量的新數據來重新訓練整個模型，這樣會導致原有的知識和能力被淹沒或是覆蓋掉，使得模型的整體表現可能會變差。因此，如果需要調整模型以適應新的任務或應用，建議使用 Fine Tuning 的方式進行微調，而不是直接進行 "bulk teaching of new knowledge"。

## 預先訓練 GPT 與 微調 GPT 有何差異

預先訓練 (pretraining) 和微調 (fine-tuning) 是兩個不同的概念。

預先訓練是指在大規模語料庫上訓練 GPT 模型，以學習通用的語言特徵和知識，如語法、語義、上下文等。預先訓練的目標是將 GPT 模型作為通用語言模型，可以用於各種自然語言處理任務，例如文本分類、語言生成、機器翻譯等。預訓練過程通常需要大量的計算資源和時間，並且使用了無標籤的大規模語料庫來訓練模型。

微調是指在預先訓練的 GPT 模型的基礎上，使用帶標籤的少量任務特定數據，對模型進行再訓練，以適應特定任務。微調的目標是提高 GPT 模型在特定任務上的性能和效果。微調通常需要較少的計算資源和時間，因為它只需要對預訓練的模型進行小規模的調整，使其更適合於特定任務。

總之，預先訓練是為了學習通用的語言知識和特徵，而微調是為了調整預先訓練的模型以適應特定任務。這兩個過程的目標和方法是不同的。

# 對於大量的 FAQ 技術文件、使用手冊、合約文件、維護紀錄、叫修紀錄等等檔案或資訊，該使用 GPT 的甚麼技術呢？

對於大量的 FAQ 技術文件、使用手冊、合約文件、維護紀錄、叫修紀錄等等檔案或資訊，可以使用 GPT 的預訓練和微調技術來建立一個自然語言處理模型，用於回答與技術相關的問題。您可以將這些文件中的文本作為訓練數據，然後通過對模型進行微調來針對您特定的應用場景進行優化。

您還可以使用內嵌技術將模型集成到您的應用程序中，以便於用戶與您的應用程序進行交互並獲取問題解答。此外，如果您的數據集很大，您還可以考慮使用 GPT-3 的 bulk teaching 技術來將新的知識直接注入模型中。






