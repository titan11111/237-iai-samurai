# 237-iai-samurai ｜ 一閃 IAI

## 一言

3D居合対戦。斬・受・崩の三すくみで読み合い、一太刀で決める。三本先取。

## 基本情報

| 項目 | 内容 |
|---|---|
| ジャンル | 3D 居合アクション（1vs1・三すくみ） |
| 構成 | `index.html` 単体（Three.js r128 CDN） |
| サイズ | 公開実体 約 0.05MB（20MB 鉄則内） |
| 画面 | iOS 75/25（`#game-shell` / `#game-stage` 75% / `#control-deck` 25%） |
| 音 | WebAudio SE。ミュートは操作盤中央。`localStorage['tg.237.muted']` |
| 保存 | `localStorage['iai.best']` に最高連勝 |

## 勝敗条件

**3本先取。** 斬りが入れば即決着。気力0で息切れ（スタン）。

## 三すくみ

| 技 | 勝ち | 負け |
|---|---|---|
| 斬 | 崩 | 受（弾き） |
| 受 | 斬 | 崩 |
| 崩 | 受 | 斬（抜き合い） |

## 操作

| 入力 | 動作 |
|---|---|
| 寄 / A / ← | 間合いを詰める |
| 退 / D / → | 間合いを取る |
| 斬 / Space / J | 斬る |
| 受 / K / Shift（押しっぱなし） | 受ける |
| 崩 / L / ↓ | 崩す |
| 操作盤 ♪ | ミュート |

## iOS対応（実装済み）

- viewport `user-scalable=no, viewport-fit=cover`
- 75/25 シェル + safe-area
- `touch-action:none` / 300ms ダブルタップ防止
- 全ボタン `pointerdown` + `setPointerCapture`（48px 以上）
- WebAudio unlock（初回タップ）
- `visibilitychange` で AudioContext 再開

## ファイル構成

- `index.html` — 本体
- `SPEC.md` / `LEARNINGS.md`

## 未確定事項

- 外部 CDN（three.js）依存。オフライン時は読み込み失敗メッセージを表示
