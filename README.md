# nadeshiko-scratch

日本語プログラミング言語「なでしこ3」を使った、小中学生向けプログラミング学習サイト（仮称: ことばブロック）。

ブロックをつないでプログラムを作り、**その隣に日本語のソースコードが常に表示され、そのテキストを直接編集するとブロックにも反映される**ブラウザ完結の学習環境を目指す。

> **開発状況**: 要件定義フェーズ完了。実装は未着手（Phase 0 のスパイクから開始）。

## ドキュメント

| 文書 | 内容 |
|---|---|
| [docs/requirements.md](docs/requirements.md) | **要件定義 v1.0**（全18章）。機能要件・非機能要件・アーキテクチャ・データモデル・法務・スパイク項目・フェーズとDoD・リスク登録簿 |
| [docs/adr/](docs/adr/) | 後戻りしにくい決定の記録（ADR 8件） |
| [docs/nako-internals.md](docs/nako-internals.md) | **なでしこ3 の非公開内部挙動への依存（I1〜I7）**。検出方法と退避先。週次 CI の契約テスト対象 |

## この製品の芯

> ブロックで作ったものが、そのまま読める日本語のプログラムになっている。
> そして、その日本語を書き換えればブロックも変わる。

## 対象

小学1年生〜中学3年生。学年モード（ひらがな／漢字+ルビ／漢字）と難易度別ブロックセットを切り替える。
学校の授業と家庭学習の両方で使える。

学習指導要領との対応: 小学校は算数5年「正多角形」（A-①）／理科6年「電気の利用」（A-②）／プログラミング入門（C-②）、中学校は技術・家庭科 技術分野 D(2) 双方向性のあるコンテンツ ／ D(3) 計測・制御。

## 技術スタック（予定）

- **エディタ**: React 19 + Vite 7 + Blockly 13.3（zelos ベース独自テーマ）+ CodeMirror 6
- **言語処理**: nadesiko3 3.8.1（pin）。逆変換は `nako_tokenizer` / `nako_lexer` / `nako_parser3` を直接束ねた自前パーサ
- **実行**: 別レジストラブルドメインの cross-origin iframe（`sandbox`、`allow-same-origin` なし、`connect-src 'none'`）
- **基盤**: Cloudflare Workers + Static Assets / D1 / R2 / Durable Objects / Queues

## ライセンス

| 対象 | ライセンス |
|---|---|
| コード | Apache-2.0（[LICENSE](LICENSE)） |
| 教材本文 | CC BY-SA 4.0（予定） |
| 同梱素材 | CC0 のみ（予定） |

**AGPL-3.0 のコード（`scratch-vm` / `scratch-gui` / `scratch-render` / `scratch-editor` / `scratch-paint`）を一行も含めない。** CI で機械的に検証する。`scratch-blocks` は Apache-2.0 なので参照可。

本プロジェクトは Scratch Foundation および なでしこ開発チームによる承認・提携を受けたものではない。
