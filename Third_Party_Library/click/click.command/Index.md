# 📢click.command()

<hr>

## 📍第1部：基礎理解

<hr>

### 📝第01章：`click.command` とは何か

* Click において CLI コマンドを定義するためのデコレータ。
* `@click.command` を付けた関数が、そのまま実行可能な CLI コマンドになる。
* 単一コマンドの CLI を作る場合の基本。


### 📝第02章：基本構文

* `@click.command()` を関数に付けると、その関数が CLI のエントリーポイントとなる。
* `python script.py` のように呼び出すと、その関数が実行される。


### 📝第03章：オプションや引数との組み合わせ

* `@click.command` と `@click.option` または `@click.argument` を組み合わせるのが典型的な使い方。
* 例として、`--name` オプションを付ければ「Hello, Alice!」のように動作を変えられる。


### 📝第04章：エイリアス（name 引数）

* `@click.command(name="hi")` のように `name` を指定すると、CLI 上での呼び出し名を変更できる。
* デフォルトでは関数名がコマンド名になる。


### 📝第05章：help 引数

* `@click.command(help="説明文")` とすると、`--help` を実行した際にその説明が表示される。
* ユーザーにとっての使いやすさを高めるために必須。


### 📝第06章：group との違い

* `click.command` は単一のコマンドを定義するためのもの。
* `click.group` は複数コマンドをまとめる親コマンドを定義する。
* 小規模 CLI は `click.command`、サブコマンドを整理したいなら `click.group` を選ぶ。

<hr>

