# Papers-compare-project-by-langflow
這次要做的是使用langflow作一個論文回覆聊天機器人，找尋不同的論文，當使用者輸入想要找的論文主題並且說明他想執行的動作 ex.比較論文書寫方式、對專有名詞的解釋差異 <br>
## 簡單的pdf回答機器人
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/simple_pdf.png)
上面的例子是一個簡單的pdf答覆聊天機器人，我們將有關一家店的所有資訊使用file以data形式output接到split text讓data轉成text chuncks. <br>
再將這些chuncks丟入vector store. <br>
### 什麼是vector stores，為甚麼要用vector stores
Vector Stores的用途是儲存和檢索由文本或其他資料產生的向量。在自然語言處理中，當你使用嵌入模型(這個例子使用OpenAI embedding)將文本轉換為向量，你需要高效的方式儲存這些向量並在需要時進行檢索。<br>
拿我的主題當作例子，當我將論文的PDF檔案轉換成向量後存入Vector DB，可以方便user查詢時根據內容相似性檢索相關的文章或段落。 <br>
以下是步驟:<br>
1. 解析PDF檔案的內容，提取出文本資料。
2. 將提取的文本通過一個嵌入模型轉換為向量。這些向量是文本的數學表示，能夠捕捉文本的語義信息。
3. 向量存入Vector DB，每篇文章或每個段落對應一個向量，可以在向量中保留相關的元數據，比如文章標題、作者、日期等。
4. 當用戶輸入查詢，查詢的輸入文字也會被轉換為向量。
5. 這個查詢向量將在Vector DB中進行比對，找到最相似的向量，並返回相關的文章或段落。
6. Vector DB會根據相似性返回最相關的結果，這些結果就是符合用戶查詢主題的文章。


