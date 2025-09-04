---
作成日時: 2025-07-24 04:31
変更日時: 木曜日 24日 7月 2025 04:31:20
所属: Topic
---

# joinメソッドの実用

---

## 1. リストの文字列結合（CSV生成）

### 解説

CSVファイルへの出力時などで、リストをカンマで連結する用途。

```python
row = ["田中", "30", "東京"]
csv_line = ",".join(row)
print(csv_line)  # → 田中,30,東京
```

---

## 2. HTMLタグ生成

### 解説

要素を `<li>` タグで囲んで、リスト全体を `<ul>` にする場面。

```python
items = ["りんご", "バナナ", "みかん"]
html = "<ul>\n" + "\n".join(f"<li>{item}</li>" for item in items) + "\n</ul>"
print(html)
```

---

## 3. 改行付きログの整形

### 解説

複数行のログやメッセージを、1つの文字列として出力する場合。

```python
logs = [
    "INFO: 開始しました。",
    "DEBUG: データを取得中。",
    "ERROR: 接続に失敗しました。"
]
formatted = "\n".join(logs)
print(formatted)
```

---

## 4. パスの構築（POSIX風）

### 解説

ファイルパスをスラッシュ区切りで組み立てる。`pathlib` が使えない環境などで有効。

```python
parts = ["home", "user", "documents", "file.txt"]
path = "/".join(parts)
print(path)  # → home/user/documents/file.txt
```

---

## 5. クエリパラメータの組み立て

### 解説

URLのGETパラメータを手動で構築する際の処理。

```python
params = {"q": "chatgpt", "lang": "ja", "limit": "10"}
query_string = "&".join(f"{k}={v}" for k, v in params.items())
url = f"https://example.com/search?{query_string}"
print(url)
```

---


