##  Python-Error&Exception-Lesson

---

### 第1部：基礎

1. **エラーと例外の違い**

   * SyntaxErrorとExceptionの区別
   * コンパイルエラーと実行時エラーの分類

2. **構文エラー（SyntaxError）の読み方**

   * `^`の位置の意味
   * よくあるパターン（カッコ、コロン、インデント）

3. **例外（Exception）の基本構造**

   * Tracebackの読み方
   * `TypeError`, `ValueError`, `NameError` の違い

4. **例外クラスの階層構造**

   * `BaseException` と `Exception`
   * `ArithmeticError`, `LookupError` 等の親クラスの存在

5. **try-except構文の基本**

   * 最低限の構文／複数except／elseとfinally

6. **組み込み例外一覧と代表的な発生要因**

   * `ZeroDivisionError`, `IndexError`, `KeyError` 等

---

### 第2部：エラーハンドリング

7. **実行時の型エラー（TypeError）の対処**

   * 関数引数／演算子の誤用パターン

8. **値に関するエラー（ValueError）の見極め**

   * `int("abc")` など／データ検証の重要性

9. **KeyErrorとIndexErrorの区別**

   * `dict`と`list`に対するアクセスの違いと注意点

10. **FileNotFoundErrorなどファイル系の例外**

    * ファイル操作における基本的なチェック手法

11. **with構文と例外処理の連携**

    * リソース解放と安全なファイル処理

12. **raiseによる例外の送出とカスタム例外の導入**

    * `raise`の書き方／`Exception`の継承

---

###  第3部：デバッグ技法

13. **Tracebackの深い読み解き方**

    * 呼び出し階層／関数名・ファイル名の確認方法

14. **例外をログ出力する技法**

    * `logging`と`traceback`モジュールの活用

15. **複雑なtry-except-else-finallyの書き方**

    * ネストと処理順序の正確な把握

16. **exceptの書き過ぎとexcept:の危険性**

    * 具体的な例外を指定すべき理由とケース

17. **複数例外をまとめて処理する**

    * `except (A, B):`構文と注意点

18. **リトライ処理とエラー回避の設計**

    * `time.sleep`との組み合わせ／`for`との統合

---

### 第4部：現場でのエラー対応

19. **バグの原因を特定するための再現コード作成**

    * ミニマルコードの作り方／最小構成の抽出

20. **エラー発生時の状態記録**

    * ログ／変数ダンプ／スナップショット出力

21. **テスト失敗時のエラー対処（pytest前提）**

    * `assert`失敗／setupエラー／例外期待テスト

22. **外部API・DBなどでのエラー処理**

    * 通信失敗／DB接続失敗時の例外の捕捉

23. **非同期処理における例外**

    * `asyncio`の例外伝播と`gather`の落とし穴

24. **エラー処理方針の決め方（例外を握る・投げる）**

    * ライブラリ開発者 vs 利用者としての責任分界

---

