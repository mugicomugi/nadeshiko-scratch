# ADR-003: Blockly 13.3.x + zelos ベースの独自テーマ

- **状態**: 提案中（Phase 0 の S-05 で一部確定）
- **日付**: 2026-09-16

## 状況

Scratch 風のブロックエディタが要る。選択肢は Blockly、scratch-blocks（Scratch Foundation フォーク）、独自実装。

## 決定

**Blockly 13.3.x ＋ zelos レンダラをベースにした独自テーマ。** Blockly 13 標準のキーボードナビゲーションとスクリーンリーダー対応を**無効化しない**。

`scratch-blocks` は Apache-2.0 なので、**レンダラとフィールドの実装を参照元として活用する**（コードのコピーではなく実装方法の参照）。

## 却下した代替案

| 案 | 却下理由 |
|---|---|
| **scratch-blocks 2.x** | `blockly ^12.4.1` に固定されており、**v13 のキーボード操作・スクリーンリーダー対応が乗らない**。学校調達でアクセシビリティ（JIS X 8341-3:2016 AA）を落とす選択はしない |
| **dnd-kit 等での独自 DnD 実装** | 接続幾何・スナップ・入れ子・undo・IME・タッチを全部自作することになり非現実的 |
| **scratch-gui のセルフホスト** | **AGPL-3.0**。Cloudflare でのホスティングは第13条に該当し全ソース開示義務。Apache-2.0 と両立しない |

## 影響

- **ブロックラベルに ruby タグが使えない**（Blockly のフィールドは SVG text）。カスタムフィールド `FieldRubyLabel`（SVG text 2本重ね）を実装する必要がある
- 独自ブロック・独自フィールドに ARIA role / label を必ず付与する（v13 リリースノートの明示要求）
- 日本語命令名＋助詞ラベルのブロック上での配置は、**英語圏の Blockly 知見が通用しない完全な未検証領域**

## 再評価トリガー

- S-05 が No-Go → `FieldRubyLabel` を諦め kana / kanji の2表記のみにする（小3〜4のブロック読解は easy vocab で担保）
- 基準端末でブロック150個のドラッグが 30fps を割る → zelos の装飾を削る → thrasos ベース → ブロック数上限（作品あたり200個）の3段階で退避

## Phase 0 で確定させること

- [ ] S-05: `FieldRubyLabel` のレイアウト崩れ、`textLength` + `lengthAdjust` でのルビ幅、ブロック150個の再レイアウト 300ms 以内、スクリーンリーダーの読み上げ、8px ルビの可読性
- [ ] S-06: Blockly の `field_input` での日本語 IME（iPad 第9世代 Safari で変換中カーソルが飛ばないか）
