## join()メソッド

`join()` メソッドは、Pythonにおいて文字列のリストなどを**特定の区切り文字で連結**するために使われます。対象は文字列の\*\*イテラブル（例：リスト、タプル）\*\*であり、各要素がすべて文字列である必要があります。

---

> ## 構文

```python
'区切り文字'.join(イテラブル)
```

---

## 基本例

```python
words = ["apple", "banana", "cherry"]
result = ", ".join(words)
print(result)  # apple, banana, cherry
```

---

## 使用例

### ① ファイルパスの連結（`pathlib`を使わない例）

```python
parts = ["home", "user", "documents"]
path = "/".join(parts)
print(path)  # home/user/documents
```

ただし、現代的には `pathlib` を使うべきです。

---

### ② HTMLの生成

```python
items = ["<li>Item1</li>", "<li>Item2</li>", "<li>Item3</li>"]
html = "<ul>" + "".join(items) + "</ul>"
print(html)
```

---

### ③ CSV形式の行を作成

```python
row = ["山田", "太郎", "30", "東京"]
csv_line = ",".join(row)
print(csv_line)  # 山田,太郎,30,東京
```

---

### ④ 各文字にスペースを挿入

```python
word = "hello"
spaced = " ".join(word)
print(spaced)  # h e l l o
```

---

### ⑤ 改行で連結（ログやテキスト生成に）

```python
lines = ["1行目", "2行目", "3行目"]
text = "\n".join(lines)
print(text)
```

---

## 注意点

* 対象のイテラブル内の要素は**すべて文字列である必要**があります。整数などが混ざっていると `TypeError` を起こします。

```python
",".join(["a", 1])  # エラー
```

対策：

```python
",".join(map(str, ["a", 1]))  # OK: "a,1"
```

---

必要に応じて、他の型（数値、辞書など）を文字列に変換してから使う場面が多く、テキスト出力やログ記録などにおいて頻出のメソッドです。
