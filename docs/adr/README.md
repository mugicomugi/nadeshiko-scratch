# Architecture Decision Records

本プロジェクトの後戻りしにくい決定を記録する。

各 ADR は **状況 / 決定 / 根拠 / 却下した代替案 / 影響 / 再評価トリガー** を持つ。
「再評価トリガー」は、その決定を見直すべき具体的な事象である。これを書かない ADR は受け入れない。

| # | 決定 | 状態 | 確定させるスパイク |
|---|---|---|---|
| [001](001-iframe-execution.md) | 実行は iframe（Web Worker ではない） | 提案中 | S-01 / S-17 |
| [002](002-guard-injection.md) | ガード注入は実行直前の AST パス | 提案中 | S-15 / S-19 |
| [003](003-blockly-13-zelos.md) | Blockly 13 + zelos ベース独自テーマ | 提案中 | S-05 |
| [004](004-alternating-single-source.md) | 編集モデルは交替単一正本 | 提案中 | S-18 / S-20 |
| [005](005-show-frame-wait.md) | 毎フレーム待 は表示用コードにも出す | 提案中 | — |
| [006](006-separate-run-domain.md) | 実行オリジンは別レジストラブルドメイン | 提案中 | S-27 |
| [007](007-no-comments.md) | 公開ギャラリーにコメント欄を実装しない | **承認済** | — |
| [008](008-nadesiko-fork-conditions.md) | なでしこ本体をフォークする条件 | 提案中 | — |

**状態**: 提案中 → 承認済 → 廃止（置き換え先を明記）
