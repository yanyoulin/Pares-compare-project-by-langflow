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
<br>
然後我們將Astra DB回傳的data用parse data轉成text，接上prompt，使它成為聊天機器人可用的店家資訊，回答使用者的問題。<br>
參考連結:https://www.youtube.com/watch?v=rz40ukZ3krQ <br>

## 開始實作-論文答覆機器人version1
架構想法:`input`-`兩個prompt分別處理 1.使用者想找的論文 2.使用者想針對這些論文作的動作`-`將論文存進Astra DB`-`將資料傳給prompt 讓機器人去分析那些文章是使用者要的`-`機器人執行第二個prompt提出的動作ex.compare`
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/project_ver1.png)
我遇到的問題: <br>
### 一個詳細且考慮所有狀況的prompt? 還是input與prompt互相配合?
原先，我很直覺的想讓OpenAI的model直接從使用者的一串句子中判斷並提取出使用者可能要的論文關鍵字以及想執行的動作，在prompt中加入few-shot learning的方法，以為這樣就能解決。<br>
原本的prompt:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/poor_prompt.png) <br>
原本的input:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/poor_input.png) <br>
原本的output:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/poor_output.png) <br>
可以發現這違反我們的意圖，但在model不完善的情況下這是可預期的結果，使用者的輸入可能每次的風格、格式、提問方法都不一樣，這造成了難以讀取。<br>
所以我決定:`讓使用者輸入是有格式的`，方便model讀取、提高正確率。<br>
修改後的prompt:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/promote_prompt.png) <br>
修改後的input:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/promote_input.png) <br>
修改後的output:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/promote_output.png) <br>
### 手動下載論文放入vector DB還是上網爬?
我的目標是想使用者輸入想要的論文關鍵字之後，app去能下載到或預覽到論文的網站直接搜索(呼叫API)，但最後version1決定先從手動下載pdf開始。<br>
先將測試用的論文利用前面提到的簡單答覆機器人的方法丟入vector DB中當作model可使用的資料，Astra DB的search input跟OpenAI分析出使用者的article相連得出我們要的資料，再用prompt告訴model從這些資料找出user要的<br>
<br>
user's input:<br>
```
Article [ReAct], Do [the definition of zero shoot in different papers]
```
Astra DB搜索結果:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1_component_output.png) <br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1_component_text.png) <br>
圖中對應到論文的段落:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1_pdf_result.png) <br>
prompt:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1_find_pdf_prompt.png) <br>
OpenAI model輸出結果:<br>
![image](https://github.com/yanyoulin/papers-compare-project-by-langflow/blob/main/langflow_project_pics/ver1_chatgptaboutpdf_output.png) <br>

結果似乎還不是我想要的，我預期是它能告訴我是哪個論文以及提取整個論文出來並準備接收使用者想對論文們做的動作(version1中尚未實作)。<br>













