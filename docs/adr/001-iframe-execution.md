# ADR-001: 実行は cross-origin iframe（Web Worker ではない）

- **状態**: 提案中（Phase 0 の S-01 / S-17 で確定）
- **日付**: 2026-09-16

## 状況

なでしこ3 は `new Function()` で JS を実行し、標準で `JS実行` 命令を持つ。ユーザーコードは常に「任意の JavaScript」として扱う必要がある。加えて Phase 4b では**他人が書いたコードを実行する**。

隔離の選択肢は2つ: cross-origin iframe か Web Worker。

## 決定

**`kotoba-run.jp` の cross-origin iframe で実行する。** `sandbox="allow-scripts allow-modals"` とし `allow-same-origin` は併用しない（opaque origin となり親の DOM / Cookie / ストレージへ到達不能）。

## 根拠

- なでしこの描画系プラグイン（`plugin_turtle`、Canvas 描画、`言`/`尋`/`二択`）は **DOM を前提としている**。Worker 版の `plugin_browser_in_worker` は Canvas / DOM / タートルを持たない
- Worker にすると描画命令を全部自作して postMessage ブリッジを書くことになり **+6週**
- `'unsafe-eval'` はなでしこ必須（外すと一切動かない）。オリジンを分けることで、その必要悪を1箇所に閉じ込められる

## 却下した代替案

| 案 | 却下理由 |
|---|---|
| **Web Worker + terminate()** | 描画命令の自作で +6週。ただし**停止の確実性は Worker のほうが上**なので、S-01（iPad でのプロセス分離）が No-Go ならここへ退避する |
| 同一オリジンの iframe | `allow-same-origin` を付けると隔離が成立しない。付けなくても Cookie の登録可能ドメイン共有問題が残る（ADR-006） |
| サーバ実行 | **技術的に不可能**。Cloudflare Workers ランタイムは `eval` / `new Function` を禁止している |

## 影響

- 実行フレームの初回転送を 150KB（brotli）以下に抑える必要がある
- postMessage プロトコルに `v: number` を必須フィールドとして持たせる（エディタと実行フレームは別々にデプロイされるためバージョンスキューが必ず起きる）
- `言` / `尋` / `二択` が cross-origin iframe で動くかは未検証（S-17）。動かない場合は `PluginKidDialog` として自前の DOM ダイアログで上書きする

## 再評価トリガー

- S-01 が No-Go（iPad で無限ループ時に親UIごとフリーズする）→ Worker + OffscreenCanvas へ全面切替、教材を2本に削って期間を維持
- I1（`asyncFn` の伝播）が壊れた → 同上

## Phase 0 で確定させること

- [ ] S-01: iPad 第9世代と Celeron N4500 Chromebook の実機で、iframe 内の無限ループ中に親UIが2秒以内に停止ダイアログを描画できるか
- [ ] S-17: cross-origin iframe（opaque origin）で `言` / `尋` / `二択` / Canvas / タートルが動くか。共有 player が top-level CSP sandbox 下で動くか
