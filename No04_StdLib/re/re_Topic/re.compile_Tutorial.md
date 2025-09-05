---
title: re.compile_Tutorial
keyword: 
created: 2025-09-05 01:05
modified: 2025-09-05 01:05
vault: Python
catergory: Language
language: Python
area: Standard_Library
identify:  
type: Topic
tole: Content
order: 00
---

## re.compile

---

### 解説

`re.compile` は、正規表現パターンを事前にコンパイルして `Pattern` オブジェクトを生成する関数です。
毎回 `re.match` や `re.search` に文字列パターンを渡すと、内部的にコンパイルが繰り返され非効率です。`re.compile` を使えばコンパイル済みオブジェクトを再利用でき、処理速度と可読性が向上します。

主な特徴:

* `Pattern` オブジェクトを返す
* `match`, `search`, `findall`, `sub` などのメソッドを持つ
* flags によって挙動を制御できる (`IGNORECASE`, `MULTILINE`, `DOTALL`, `VERBOSE` 等)

---

### 書式

```python
import re
pattern = re.compile(pattern, flags=0)
```

* `pattern`: 正規表現の文字列（raw文字列 `r"..."` を推奨）
* `flags`: オプションフラグ。省略時は `0`

---

### 使用例

```python
import re

# 数字の連続にマッチするパターンをコンパイル
pat = re.compile(r"\d+")

# match（先頭のみ）
m = pat.match("123abc")
print(m.group())   # 123

# search（全文から最初の一致を探す）
s = pat.search("abc123xyz")
print(s.group())   # 123

# findall（すべての一致をリストで返す）
print(pat.findall("foo123bar456"))  # ['123', '456']

# sub（置換）
print(pat.sub("#", "foo123bar456"))  # foo#bar#
```

---

### 応用例

#### flagsの活用

```python
p = re.compile(r"abc", re.IGNORECASE)
print(p.match("ABC"))  # 大文字でもマッチ
```

#### VERBOSEモードでの可読化

```python
email = re.compile(r"""
    (?P<user>[\w.+-]+)   # ユーザー名
    @
    (?P<domain>[\w.-]+)  # ドメイン
""", re.VERBOSE)

m = email.match("foo.bar@example.com")
print(m.groupdict())  # {'user': 'foo.bar', 'domain': 'example.com'}
```

---

### 注意点

* 複雑なパターンは可読性を損なうため `VERBOSE` フラグで整理すべき
* 巨大テキストに正規表現を多用すると性能問題になりやすい
* `re` モジュールは直呼びでもキャッシュするが件数制限があるため、頻用パターンは `compile` を明示的に使う方が安全

---

