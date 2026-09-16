# なでしこ3 への依存面（契約）

本プロジェクトは なでしこ3 の**公開APIとして保証されていない内部挙動**に依存している。
依存面は I1〜I7 の7項目。本書はその一覧と、破壊されたことを検出する方法、破壊されたときの退避先を記録する。

- 対象バージョン: **`nadesiko3@3.8.1`**（npm で pin）
- 接触を許すパッケージ: **`@nb/nako-parser` と `@nb/nako-adapter` の2つのみ**。他パッケージからの `nadesiko3` の import は ESLint (`import/no-restricted-paths`) で禁止する
- `nadesiko3core` の npm 版は 3.6.22 / 2024-09 で更新停止しているため**直接依存しない**

> **なぜ本書が要るか**: なでしこ3 は kujirahand 氏の単独開発（open issues 223件、月1〜2回リリース）であり、3.7系で実際に破壊的変更があった（`__getProp`/`__setProp` の廃止、`@` の優先度変更、定数の書き換え禁止）。ここに挙げた7項目は**作者が本プロジェクトの用途を想定して保証したものではない**。壊れることを前提に、検出方法と退避先を先に決めておく。

---

## 依存面の一覧

| # | 依存する内部詳細 | 公開API保証 | 使う機能 | 検出方法 | 退避先 |
|---|---|---|---|---|---|
| **I1** | `asyncFn` の伝播意味論（`usedAsyncFn` / `topOfFunctionAsync`） | なし | FrameScheduler 全体 | 生成JSに `async function` と `await` が出ることの文字列検査 | Web Worker + `terminate()`（+6週） |
| **I2** | `には` 構文の `func_obj` ノード形状 | なし | イベントハット・`ずっと繰り返す` | AST スナップショット | 独自構文を作らず `def_func` で代替 |
| **I3** | `__nako_make_closure` のクロージャ挙動 | なし | スプライトごとの状態 | S-16 の並行テスト | プラグイン側の `Map<spriteId, state>` |
| **I4** | `nako_parser3` の AST NodeType（全52種） | なし | **逆変換の基盤** | `parser_ast_golden.json` の差分 ＋ 自前 `canonAst` ゴールデン | 対応表の見直し（年1〜2回、3〜5人日） |
| **I5** | `nako_gen` の `convTryExcept` が素の try/catch であること（再 throw フィルタがない） | なし | ガードの再 throw 設計 | E2E（`エラー監視` で囲んだ無限ループが 300ms で止まる） | `useDebug` 常時へ退避 |
| **I6** | `sys.__findFunc` | なし | `には` コールバックの受け取り | 単体テスト | 関数オブジェクトを直接渡す |
| **I7** | **`__nako_scope_enter` / `__nako_leave` と issue #1758 の `__self.__vars` 待避・復元が、複数 async スレッドのインターリーブ下でも正しいこと** | なし | **並行実行全体** | **S-16**（1000フレーム × 10体の決定的検証） | スプライト状態をプラグイン側に持つ |

**I7 が最も危険**: なでしこのスコープ実装は単一スタック前提であり、issue #1758 のパッチに全面依存している。作者が並行実行を想定して保証したものではない。Phase 0 の S-16 で決定的に検証すること。

---

## パーサ側で依存している具体的な挙動

逆変換エンジン（`@nb/nako-parser` / `@nb/decompiler`）は、上記とは別に以下の**具体的な実装事実**に依存する。いずれも一次情報として確認済み。

### P1. `end:` の SourceMap は「次のトークン」を指す

`nako_parser3.mts` の `end:` 設定 **70箇所すべて**が引数なしの `peekSourceMap()` を呼んでいる。これは現在のノードの終端ではなく**次のトークン**の SourceMap である。

### P2. `yCallFunc` の `startOffset` は動詞トークンの位置

関数呼び出しノードは map を動詞トークンの位置で取る。**日本語は引数が動詞より前に来る**ため、`node.startOffset` から `node.end.endOffset` を切り出すと**引数が範囲の外に出る**。

> したがって `code.slice(node.startOffset, node.end.endOffset)` は使ってはならない。原文範囲は**部分木走査 `[min(descendant.startOffset), max(descendant.endOffset)]` と、その範囲に含まれるトークン index 範囲**から算出する。文レベルの `raw_nako` は常に行境界へスナップする。（要件定義 §10.4）

### P3. `funclist` はユーザー定義関数で汚染される

`lex()` → `parse()` を繰り返すと `funclist` にユーザー定義関数が蓄積し、**以後の AST が変わる**。呼び出しごとに `reset({ needToClearPlugin: false })` を実行すること。

### P4. `addFunc` ではシグネチャを完全に再現できない

`addFunc` は `isVariableJosi` / `funcPointers` を設定できないため、可変長助詞の命令（`連続表示`、`連結`、`文字列連結`）の arity が実行側とずれる。

> 上流プラグインから `meta`（`josi` / `isVariableJosi` / `funcPointers` / `varnames` / `return_none` / `asyncFn` / `pure`）ごとコピーし、`fn` だけを NOOP に差し替えた「実装なしプラグインオブジェクト」を `addPluginObject` で登録すること。

### P5. `useBasicPlugin: false` でもバンドルは軽くならない

`plugin_system` は `useBasicPlugin: false` にしてもバンドルから落ちない。「98KB 削減」「エディタオリジンに危険命令の実装が一切存在しない」という主張は**成立しない**。

> 逆変換用パーサは `NakoCompiler` を使わず `nako_tokenizer` / `nako_lexer` / `nako_parser3` を**直接束ねる**。CI でバンドル内容を assert し、`plugin_system` / `plugin_browser` / `nako_gen` / `nako_runner` が含まれないことを検証する。得られる利得は「バンドルが軽くなる」ではなく「**`'unsafe-eval'` が不要になる**」である。

### P6. `取込`（require）は `parse()` に渡すと落ちる

`取込` を含むコードは `kind: 'lex'` のエラーで落ちるため、`parse()` に渡す前に `listRequireStatements(tokens)` で検査し、存在したら `kind: 'denied'` ＋ span 付きで即返すこと。

### P7. コメントは AST に残らない

コメントは `parse()` の AST ではなく `lex()` の `commentTokens` から取得し、`code.slice(startOffset, endOffset)` で記法込み原文を保持する。付着規則は 行末 → その文 ／ コメント行 → 次の文（`placement: 'above'`）／ 孤立 → ワークスペースコメント。

### P8. `ずっと繰り返す` という構文は存在しない

予約語38語に「ずっと」はない。素の `ずっと繰り返す` は**必ず構文エラーになる**。プラグイン命令として定義し `には` 構文で本体を受ける（`ずっと繰り返すには … ここまで`）。

### P9. 合成トークンは `startOffset` が null

シンタックスハイライトで `lex()` のトークン列を装飾に変換する際、`startOffset` が null の合成トークンはスキップすること。

---

## テストコーパスの入手先

| コーパス | 場所 | 注意 |
|---|---|---|
| `parser_corpus.mjs` | npm 配布物に含まれる | 実測 **67エントリ** |
| `parser_ast_golden.json` | npm 配布物に含まれる | 154,953 バイト |
| `core/test/fixtures/compat/` | **GitHub master にのみ存在**（npm 配布物に含まれない） | 固定コミットSHAで vendoring し `corpus/SOURCE.json` に記録する（MIT 帰属表示）。形式は `{group, description, cases[]}` |

---

## フォークする条件（ADR-008 に対応）

なでしこ本体をフォークすると、バス係数1のプロジェクトを引き取ることになり保守不能になる。原則として**フォークしない**。上流 PR で解決する。

ただし以下のいずれかに該当したら、**フォーク判断会議を開く**:

- (a) 作者の活動が **6ヶ月停止**した
- (b) **I1〜I7 のうち2つ以上が同時に壊れた**
- (c) セキュリティ上の問題に **4週間対応がない**

## 上流への貢献（関係構築として先に行う）

1. `nako_gen` への**軽量ガードオプション PR**（`await` / `postMessage` を伴わず `if (__v0.get('__forceClose')) return -1;` のみを挿入する、十数行）
2. **`asyncFn` 伝播と `func_obj` の挙動を doc 化する PR**

ドキュメント化されれば実質的に安定APIになり、破壊的変更時に気付いてもらえる。名称・ロゴの相談も同時に行う。

## 週次 CI

`nadesiko3@latest` に対する golden 実行 ＋ I1〜I7 それぞれの契約テスト。
**失敗しても `main` は赤くせず、Issue を自動起票して追随コストを可視化する。**
