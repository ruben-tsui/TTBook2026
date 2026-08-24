# 正規表示式語料庫檢索：20 個單語實用範例 -英文・中文（臺灣 傳統漢字）

> 本版專注於單語英語語料庫與單語中文語料庫的搜尋、分析、清理與品質檢查。
> 正規表示式方言：PCRE2 + Unicode。優先使用 `\p{Han}`、`\p{Latin}`、`\p{Lu}` 等 Unicode Property，但文本若為純英文，則不必使用 Unicode Property。在純英文語料庫中，\w 已足夠代表英文字母。

**核心觀念**：
> regex 的目的不是「找到一個字」，而是把語言資料中的形式變體、句法結構、篇章模式、格式異常與可重複的文本現象變成可檢索的條件。

**關於 `( )` 與 `(?:)` 的說明**：
本講義中所有群組皆使用 `( )`。`( )` 為捕獲組 (capturing group)，會產生 `\1`、`\2` 等編號；`(?: )` 為非捕獲組 (non-capturing group)，只分組不產生編號。在僅需分組而不需要替換或回溯的情況下，兩者功能相同。為避免初學階段增加記憶負擔，本講義統一使用 `( )`，捕獲結果直接捨棄。僅在需要 `\1` 回溯 (backreference) 時（如重複詞檢查），才需留意編號。

**關於 `\b` 的說明**：
`\b` 標示詞邊界，僅在英文中用於避免子字串匹配，例如 `ability` 與 `disability`、`capability` 、`responsibility` 同時出現時，`\bability\b` 只匹配完整詞。為避免版面干擾，以下範例中的正規表示式省略 `\b`，實際應用時可依需要自行加上。中文不使用 `\b`，因為中文沒有詞邊界(word boundary) 。

---

## 總覽

| # | 語言 | 等級 | 主題 | 核心語法 | 語料庫應用 |
|---|------|------|------|---------|------------|
| 1 | 英文 | 🟢初階 | 詞形變化 run / runs / ran / running | `( )` | 詞形檢索 |
| 2 | 英文 | 🟢初階 | 否定縮寫 don't / can't | `['’]`、`( )` | 口語／網路文本 |
| 3 | 英文 | 🟢初階 | 英式／美式拼法 color / colour | `?` | 版本差異 |
| 4 | 英文 | 🟢初階 | 疑問句開頭 | `^`、`( )` | 句型統計 |
| 5 | 英文 | 🟢初階 | 大寫縮寫詞 USA / NLP | `\p{Lu}` | 專有名詞／術語 |
| 6 | 英文 | 🟡中階 | not 結構 | `( )`、`\s+` | 否定模式 |
| 7 | 英文 | 🟡中階 | 程度副詞 + adjective | `( )`、`\s+` | 強度／搭配 |
| 8 | 英文 | 🟡中階 | there is / there are | `( )` | 句法模式 |
| 9 | 英文 | 🟡中階 | 句末標點 + 引號 | 字元類別 | 編輯 QA |
| 10 | 英文 | 🟡中階 | 重複詞 the the | `\1` | OCR／打字錯誤 |
| 11 | 英文 | 🔴高階 | 片語動詞 look up 分離與連接 | `( )`、量詞 | 詞彙模式 |
| 12 | 英文 | 🔴高階 | 關係子句標記 | `( )` | 句法檢索 |
| 13 | 中文 | 🟢初階 | 中文標點 | 標點字元 | 排版 QA |
| 14 | 中文 | 🟢初階 | 中文數字 | 字元類別 | 數字表示法 |
| 15 | 中文 | 🟡中階 | 中文重複詞 | `(\p{Han}{1,4})\1` | OCR／編輯 QA |
| 16 | 中文 | 🔴高階 | 臺灣日期格式 | alternation | 時間正規化 |
| 17 | 中文 | 🔴高階 | 臺灣地址與電話 | lookaround | 結構化文本 |

---

# 🟢 第一部分：英文 — 初階

## 1. 動詞詞形：`run / runs / running / ran`

- **測試文字**：
  ```text
  The program runs every day.
  The researchers ran the experiment twice.
  We are running a new study.
  The system can run automatically.
  A runner is waiting outside.
  ```
- **正規表示式**：
  ```regex
  (ran|run(s|ning)?)
  ```
  明列寫法：
  ```regex
  (run|runs|ran|running)
  ```

## 2. 否定縮寫：`don't / can't / won't / isn't`

- **測試文字**：
  ```text
  I don't know the answer.
  She can't attend the meeting.
  We won't change the policy.
  The result isn't surprising.
  They haven't finished yet.
  ```
- **正規表示式**：
  ```regex
  (don|can|won|isn|haven|hasn)['’]t
  ```

## 3. 英式／美式拼法：`color / colour`

- **測試文字**：
  ```text
  The color of the interface was changed.
  The colour scheme is easy to read.
  Colors may vary between devices.
  discolor is not a target.
  ```
- **正規表示式**：
  ```regex
  colou?rs?
  ```
- **不區分大小寫**：
  ```regex
  (?i)colou?rs?
  ```

## 4. 疑問句開頭：`^`

- **測試文字**：
  ```text
  What does the corpus contain?
  How many texts were included?
  Did the participants agree?
  Whatever the result, we continue.
  ```
- **正規表示式**：
  ```regex
  ^(What|How|Why|When|Where|Who|Which|Do|Does|Did|Is|Are|Was|Were)
  ```
  需搭配多行模式 `m`。

## 5. 大寫縮寫詞：`USA / UNESCO / NLP`

- **測試文字**：
  ```text
  The study was conducted in the USA.
  UNESCO published the report.
  NLP methods were used for analysis.
  ```
- **正規表示式**：
  ```regex
  \p{Lu}{2,}
  ```

---

# 🟡 第二部分：英文 — 中階與高階

## 6. 否定結構

- **測試文字**：
  ```text
  The method is not reliable.
  The participants did not agree.
  ```
- **正規表示式**：
  ```regex
  not
  (is|are|was|were|do|does|did|have|has|had|may|might|can|could|will|would|should)\s+not
  ```

## 7. 程度副詞 + adjective

- **測試文字**：
  ```text
  very useful
  highly effective
  every useful
  ```
- **正規表示式**：
  ```regex
  (very|highly|extremely|particularly|relatively|surprisingly)\s+\p{L}+
  ```

## 8. Existential there

- **測試文字**：
  ```text
  There is a problem with the data.
  There are several possible explanations.
  Therefore we continue.
  ```
- **正規表示式**：
  ```regex
  there\s+(is|are|was|were)
  ```

## 9. 句末標點 + quotation mark

- **測試文字**：
  ```text
  She said, "This is important."
  This is important."
  ```
- **正規表示式**：
  ```regex
  [.!?]["'’”]?\s*$
  ```

## 10. 重複詞

- **測試文字**：
  ```text
  The the method is very useful.
  This is is a test.
  ```
- **正規表示式**：
  ```regex
  (\w+)\s+\1
  ```

## 11. 片語動詞：`look up`

- **情境**：`look up` 可連接或分離，`look up something` 與 `look something up` 皆可能，`something` 為 1 至 5 個詞。
- **測試文字**：
  ```text
  Please look up the word in the dictionary.
  Please look the word up in the dictionary.
  We need to look up the corpus data from last year.
  We need to look the corpus data from last year up.
  I will look up the information.
  I will look the information up.
  ```
- **正規表示式（1）連接：look up + 受詞**：
  ```regex
  look up (\w+(\s+\w+){0,4})
  ```
- **正規表示式（2）分離：look + 受詞 + up**：
  ```regex
  look (\w+(\s+\w+){0,4}) up
  ```
- **正規表示式（合併，兩種順序皆接受）**：
  ```regex
  look (up (\w+(\s+\w+){0,4})|(\w+(\s+\w+){0,4}) up)
  ```
- **說明**：`(\w+(\s+\w+){0,4})` 表示 1 至 5 個詞，`\w+` 為一個詞，`(\s+\w+){0,4}` 為後續 0 至 4 個詞。

## 12. 關係子句標記

- **測試文字**：
  ```text
  The book that I read was interesting.
  The person who came yesterday left early.
  ```
- **正規表示式**：
  ```regex
  (who|whom|whose|which|that)
  ```

---

# 🟢 第三部分：中文 — 初階

## 13. 中文標點檢查

- **測試文字**：
  ```text
  這是全形標點，。！？
  這是混用,.!?
  ```
- **正規表示式**：
  ```regex
  [，。！？；：、（）「」『』【】《》〈〉]
  ```

## 14. 中文數字

- **測試文字**：
  ```text
  三個人
  第一百二十三頁
  ```
- **正規表示式**：
  ```regex
  [零一二三四五六七八九十百千萬億〇兩]{1,8}
  ```

---

# 🟡 第四部分：中文 — 中階

## 15. 中文連續重複

- **測試文字**：
  ```text
  這是非常非常重要的結果。
  我們我們需要重新檢查資料。
  營營役役的生活中，偶爾也需要休息。
  這個問題需要思考思考。
  ```
- **正規表示式**：
  ```regex
  (\p{Han}{1,4})\1
  ```
- **說明**：此模式除了抓取 `非常非常`、`我們我們` 這類因輸入或 OCR 產生的重複，也可找到 `曡字／曡詞` 現象，如 `營營役役`、`思考思考` 等。是否屬於錯誤或修辭手法，需依上下文判斷。


## 16. 臺灣常見日期格式

- **測試文字**：
  ```text
  公告日期：2026/08/24
  發布日期：2026-08-24
  會議日期：2026年8月24日
  民國115年8月24日正式生效。
  ```
- **正規表示式**：
  ```regex
  (19|20)\d{2}[/-]\d{1,2}[/-]\d{1,2}|(19|20)\d{2}年\d{1,2}月\d{1,2}日|民國\s*\d{1,3}年\d{1,2}月\d{1,2}日
  ```

## 17. 臺灣地址與電話

- **測試文字**：
  ```text
  聯絡電話：02-23661234
  手機：0912-345-678
  地址：臺北市大安區羅斯福路四段一號
  地址：新北市板橋區文化路一段100號
  地址：臺北市大安區羅斯福路四段1巷2弄3號
  地址：高雄市前鎮區中山二路100號之2號
  ```
- **電話候選**：
  ```regex
  (?<!\d)(0\d-\d{6,8}|09\d{2}-\d{3}-\d{3})(?!\d)
  ```
- **地址候選（修正版）**：
  ```regex
  [\p{Han}]{2,6}[市縣][\p{Han}]{1,8}[區鎮鄉市][\p{Han}]{1,12}([路街]|大道)([\p{Han}\d]+段)?([\p{Han}\d]+巷)?([\p{Han}\d]+弄)?[\p{Han}\d]+(之[\p{Han}\d]+)?號
  ```
  若需分步閱讀：
  ```regex
  [\p{Han}]{2,6}[市縣]                # 縣市，如 臺北市、新北市
  [\p{Han}]{1,8}[區鎮鄉市]            # 區鎮鄉市，如 大安區、板橋區
  [\p{Han}]{1,12}([路街]|大道)        # 路街或大道，道路名稱
  ([\p{Han}\d]+段)?                  # 段，如 四段、1段，段為獨立單位，不與路街並列
  ([\p{Han}\d]+巷)?                  # 巷，如 1巷、一巷
  ([\p{Han}\d]+弄)?                  # 弄，如 2弄、二弄
  [\p{Han}\d]+(之[\p{Han}\d]+)?號   # 號，如 一號、100號、100號之2號
  ```
- **說明**：
  - `[市縣]`、`[區鎮鄉市]`、`[路街]` 為單字元選項的字元類別寫法。
  - `([路街]|大道)` 中 `大道` 為雙字元詞，必須使用 `(A|B)` 形式。
  - `段`、`巷`、`弄` 為不同層級的地址單位，不應與 `[路街]` 並列為同一選項，改為各自可選的 `(\d+段)?`、`(\d+巷)?`、`(\d+弄)?`。
  - `[市|縣]` 這類寫法會匹配字元 `|`，為常見錯誤，應寫為 `[市縣]`。

---

# 附錄

## 字元類別 `[]` 與選項 `(A|B)` 的區別

- `[AB]`：一個字元，A 或 B。`[市|縣]` 會匹配 `|`。
- `(A|B)`：字串 A 或字串 B，可為多字元，如 `大道`。
- 單字元選項優先使用 `[]`：`[市縣]`、`[區鎮鄉市]`、`[路街]`。
- 含多字元選項必須使用 `( )`：`([路街]|大道)`。

## 為什麼以 `\p{Han}` 為主？

`\p{Han}` 表示 Unicode Han Script 字元，為基於標準字元屬性的描述。舊式寫法 `[一-龥]` 為人工列舉範圍，覆蓋範圍較窄。

---
PCRE2/Unicode 為基礎；中文部分僅使用 `( )`；`\b` 已從範例中省略，必要時可自行加上。*