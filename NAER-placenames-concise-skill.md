# NAER 外國地名譯名 → DSL (Concise)

Downloads the **國家教育研究院-外國地名譯名** dataset and produces a single `.dsl` file. No Parquet, no dictzip compression.

## Output

`外國地名譯名.dsl` — UTF-16 LE Abbyy Lingvo DSL dictionary.

---

## Step 1 — Install Python packages

```bash
pip install pandas odfpy
```

## Step 2 — Download source data

Search URL: `https://terms.naer.edu.tw/download/1/?refine=1&query_term=外國地名譯名`

Direct URL (may change if the site reorganises):

```bash
curl -o 外國地名譯名壓縮檔_VrsZ2AD.zip \
  "https://terms.naer.edu.tw/media/terms_data/1/%E5%A4%96%E5%9C%8B%E5%9C%B0%E5%90%8D%E8%AD%AF%E5%90%8D%E5%A3%93%E7%B8%AE%E6%AA%94_VrsZ2AD.zip"
unzip 外國地名譯名壓縮檔_VrsZ2AD.zip -d 外國地名譯名壓縮檔_VrsZ2AD
```

Expected: `外國地名譯名壓縮檔_VrsZ2AD/` containing `_0.ods` through `_6.ods`.

## Step 3 — Convert ODS → DSL

Save this as `placenames_to_dsl.py`:

```python
#!/usr/bin/env python3
import html, re, argparse, pandas as pd
from pathlib import Path

def escape_dsl(text):
    if text is None: return ""
    text = str(text).replace("\\", "\\\\").replace("[", "\\[").replace("]", "\\]").replace("\t", " ")
    return text

def clean_text(text):
    if text is None: return ""
    text = str(text).strip()
    if text.lower() == "nan" or text == "": return ""
    text = text.replace("_x000D_", "").replace("\r\n", "\n").replace("\r", "\n")
    text = html.unescape(text)
    text = re.sub(r"<[^>]+>", "", text).strip()
    return text

def convert(input_dir, output_file):
    rows = []
    for f in sorted(Path(input_dir).glob("外國地名譯名壓縮檔_?.ods")):
        df = pd.read_excel(f, engine="odf")
        rows.extend(df.to_dict("records"))
        print(f"Read {len(df)} rows from {f.name}")
    print(f"Total rows: {len(rows)}")
    with open(output_file, "w", encoding="utf-16") as f:
        f.write("# -*- coding: utf-16 -*-\n")
        f.write("#NAME \"外國地名譯名(En-Zh) 版本：2026\"\n")
        f.write("#INDEX_LANGUAGE \"English\"\n")
        f.write("#CONTENTS_LANGUAGE \"Chinese\"\n")
        f.write("\n")
        for row in rows:
            en = clean_text(row.get("英文名稱", ""))
            zh = clean_text(row.get("中文名稱", ""))
            if not en and not zh: continue
            if zh: f.write(f"{escape_dsl(zh)}\n")
            if en and en != zh: f.write(f"{escape_dsl(en)}\n")
            f.write(f"    {escape_dsl(en)}\n")
            f.write(f"    {escape_dsl(zh)}\n")
            for col, label in [("所在國","所在國："), ("經緯坐標","經緯坐標："),
                               ("備註","備註："), ("更新日期","更新日期：")]:
                val = clean_text(row.get(col, ""))
                if val: f.write(f"    [c maroon]{label}[/c]{escape_dsl(val)}\n")
            eid = row.get("ID")
            if eid is not None and str(eid).strip():
                f.write(f"    [c maroon]ID: [/c]{eid}\n")
            f.write("\n")
    print(f"DSL written to: {output_file}")

if __name__ == "__main__":
    p = argparse.ArgumentParser()
    p.add_argument("--input-dir", default="外國地名譯名壓縮檔_VrsZ2AD")
    p.add_argument("--output", default="外國地名譯名.dsl")
    args = p.parse_args()
    convert(args.input_dir, args.output)
```

```bash
python3 placenames_to_dsl.py --input-dir 外國地名譯名壓縮檔_VrsZ2AD --output 外國地名譯名.dsl
```

---

## DSL structure

- Double headwords: Chinese name (line 1), English name (line 2)
- Body fields indented, labels in `[c maroon]` markup
- Columns: 英文名稱, 中文名稱, 所在國, 經緯坐標, ID, 備註, 更新日期
