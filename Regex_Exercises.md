# 進階實戰練習題組與詳解

## 作業與實作測驗

請使用支援 **PCRE2 / Unicode** 的編輯器（如 **Zed**、**VS Code**、**CudaText**、**Python regex 模組**或 **Calibre**）完成以下 **5** 道翻譯科技實務練習題。

## 練習一（基礎題）：中文譯文標點符號 QA（Punctuation QA）

### 情境說明

譯者交出的繁體中文譯文中混雜了半形括號 `()` 與半形冒號 `:`。請撰寫 PCRE2 regex 找出所有包覆中文漢字的「半形括號」，以及緊接在中文漢字後方的「半形冒號」。

### 測試文本

```text
專案名稱(翻譯科技簡介): 本章節探討雙語語料對齊(Alignment)與自動品質檢查:QA機制。
```

### 參考解答與模式

題目 1（包覆中文的半形括號）：

```regex
\((?=\p{Han})\p{Han}+\)
```

題目 2（中文漢字後的半形冒號）：

```regex
(?<=\p{Han}):
```

### 預期輸出

- 題目 1 成功匹配 `(翻譯科技簡介)`，排除 `(Alignment)`。
- 題目 2 成功匹配 `檢查` 後之半形冒號，排除時間或英文冒號。

### 詳細解析

題目 1 利用 Lookahead `(?=\p{Han})` 確保開口括號後緊接中文；題目 2 利用 Positive Lookbehind `(?<=\p{Han})` 鎖定漢字後之冒號。

---

## 練習二（基礎題）：歐洲多語專有名詞比對（European Name Extraction）

### 情境說明

某旅遊與法律對照語料庫中包含法文、德文、冰島文地名與品牌名。請撰寫 PCRE2 regex 模式，完整擷取句子中所有含歐洲變音字元的單字。

### 測試文本

```text
The delegation traveled from Zürich and Genève to Hótel Frón in Reykjavík.
```

### 參考解答與模式

模式：

```regex
\b(?=\p{Latin}*[\P{ASCII}])\p{Latin}+\b
```

或集合交集模式：

```regex
\b\p{Latin}*[\p{Latin}&&[^\x00-\x7F]]\p{Latin}*\b
```

### 預期輸出

```text
Zürich, Genève, Hótel, Frón, Reykjavík
```

### 詳細解析

`\p{Latin}+` 能抓取所有拉丁單字；搭配 Lookahead `(?=\p{Latin}*[\P{ASCII}])` 確保單字中至少含有一個非 ASCII 變音字元，自動過濾掉純英文字詞。

---

## 練習三（中階題）：雙語對照標題順序對調與括號規格化（Bilingual Reordering）

### 情境說明

從舊系統導出的詞彙表格式為「中文詞條 (English Term)」，現需依客戶 Style Guide 統一改為「English Term（中文詞條）」（英文在前、中文在後，並改用全形括號）。

### 測試文本

```text
1. 機器翻譯 (Machine Translation)
2. 自然語言處理 (Natural Language Processing)
3. 電腦輔助翻譯 (Computer-Assisted Translation)
```

### 參考解答與模式

搜尋模式：

```regex
(\p{Han}+)\s*[(（](\p{Latin}+(?:\s+\p{Latin}+)*)[)）]
```

替換模式：

```regex
$2（$1）
```

### 預期輸出

```text
1. Machine Translation（機器翻譯）
2. Natural Language Processing（自然語言處理）
3. Computer-Assisted Translation（電腦輔助翻譯）
```

### 詳細解析

第一組 `(\p{Han}+)` 捕獲中文漢字，`[(（]` 與 `[)）]` 同時相容全形與半形括號，第二組捕獲英文多字術語。替換為 `$2（$1）` 完成順序重排與括號規格化。

---

## 練習四（中階題）：在地化數字與單位間距 QA（Numerical Spacing QA）

### 情境說明

依據英文在地化規範，數值與單位（如 `GB`、`MB`、`km`、`kg`）之間除 `%` 外均應保留一半形空格。請撰寫 PCRE2 regex 找出所有數值後方緊接單位卻缺少空格的格式錯誤（例如 `50GB`、`12.5km`）。

### 測試文本

```text
The file size is 50GB, the distance is 12.5km, and memory usage is 85%.
```

### 參考解答與模式

搜尋模式：

```regex
(\p{Nd}+(?:\.\p{Nd}+)?)(?=(?:GB|MB|KB|km|kg|cm)\b)
```

替換模式：

```regex
$1 
```

> 在 `$1` 後方加入一半形空格。

### 預期輸出

```text
The file size is 50 GB, the distance is 12.5 km, and memory usage is 85%.
```

### 詳細解析

`(\p{Nd}+(?:\.\p{Nd}+)?)` 捕獲整數或小數；正前查找 `(?=(?:GB|MB|KB|km|kg|cm)\b)` 鎖定目標單位，且自動排除不需空格的 `%` 符號。

---

## 練習五（高階題）：網頁爬蟲雙語對齊語料前處理（Web Crawl Data Sanitization）

### 情境說明

從網頁爬取的雙語句對中夾雜了 HTML 標籤、注音雜訊、重複全形標點（如 `！！`）以及「英文（中文）」格式，請設計組合模式完成清洗。

### 測試文本

```text
<p>第一條：版權所有ㄅㄆㄇㄈ！！ <span>Natural Tea Manor（天然茶莊）</span> provides fine tea.</p>
```

### 參考解答與模式

步驟 1（去 HTML 標籤）：

```regex
<[^>]+>
```

替換為空。

步驟 2（清注音與重複標點）：

```regex
\p{Bopomofo}+|(\p{P})\1+
```

替換為：

```regex
$1
```

步驟 3（雙語重排）：

```regex
(\b\p{Latin}+(?:\s+\p{Latin}+)*)\s*[(（](\p{Han}+)[)）]
```

替換為：

```regex
$2（$1）
```

### 預期輸出

```text
第一條：版權所有！ 天然茶莊（Natural Tea Manor） provides fine tea.
```

### 詳細解析

綜合運用 `\p{Bopomofo}`、標點回參 `(\p{P})\1+` 去重、以及雙語重排，展示了 Unicode Script 與 Property 在語料前處理上的完整管線。
