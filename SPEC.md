# 237-iai-samurai ｜ 一閃 IAI

## 一言

3D居合対戦。斬・受・崩の三すくみで読み合い、一太刀で決める。三本先取。

## 基本情報

| 項目 | 内容 |
|---|---|
| ジャンル | 3D 居合アクション（1vs1・三すくみ） |
| 構成 | `index.html` 単体（Three.js r128 CDN） |
| サイズ | 公開実体 約 5.6MB（20MB 鉄則内）。**実際の通信は 1.4MB**（m4aだけを取得し mp3 は落とさない） |
| 画面 | iOS 75/25（`#game-shell` / `#game-stage` 75% / `#control-deck` 25%） |
| 音 | WebAudio SE ＋ BGM（`Steel_in_the_Grass.m4a` 2分59秒・HE-AAC 64k・1.4MB）。ミュートは操作盤の上段中央。`localStorage['tg.237.muted']` |
| 保存 | `localStorage['iai.best']` に最高連勝 |

## 勝敗条件

**3本先取。** 斬りが入れば即決着。気力0で息切れ（スタン）。

## 三すくみ

| 技 | 勝ち | 負け |
|---|---|---|
| 斬 | 崩 | 受（弾き） |
| 受 | 斬 | 崩 |
| 崩 | 受 | 斬（抜き合い） |

## 舞台と登場人物（2026-09-16 追加）

| 要素 | 内容 |
|---|---|
| 顔 | 眉・白目・瞳（accent色で発光）・鼻筋・口・顎当て。頭の球面へ極座標で配置する `onFace()` で組む |
| 鉢巻 | accent色。紅＝己／藍＝敵の識別。結び目と垂れは `hachi` グループごと揺れ、斬撃で跳ねる |
| 応援者 | 輪（半径5.28）の外、半径7.4の奥側の弧に5人。立3（扇2・幟1）＋正座2（置き提灯） |
| 幟 | canvas製テクスチャに「一閃」。中央やや左の奥に立てる |
| 歓声 | `hype`（0〜1）。決着で1になり毎秒0.45で減衰。腕の上がり・跳ね・幟の揺れの振幅を共有する |

応援者は影を落とさない（描画コストを上げないため）。縦持ちの画角は横に狭いので、
5人とも `|x| ≤ 4.5` に収めている。

## BGM（2026-09-16 追加）

| 項目 | 内容 |
|---|---|
| 実体 | `Steel_in_the_Grass.m4a`（HE-AAC 64kbps・1.43MB）をループ再生 |
| 保険 | `Steel_in_the_Grass.mp3`（192kbps・4.11MB）を `<source>` の2番目に置く。HE-AAC対応ブラウザは**m4aだけを取得し、mp3は1バイトも落とさない**（実測で確認） |
| 変換 | `afconvert -f m4af -d aach -b 64000 -s 3 in.mp3 out.m4a`（macOS標準。ffmpegの内蔵aacはHE-AACを出せない） |
| 読み込み | `preload="none"`。**起動時に取りに行かない**（起動時に音声を掴むとページ読み込み完了が来ず、harness の `networkidle` が20秒で切れる） |
| 開始 | 最初のタップ／キー入力（`unlockAudio()`）。iOSの自動再生制限に従う |
| 音量 | `createMediaElementSource` → `GainNode` 経由で 0.38。1.4秒でフェードイン |
| ミュート | ♪ボタンと連動。0.32秒でフェードアウトしてから `pause()` |
| 復帰 | `visibilitychange`。裏へ回ったら止め、戻ったら続きから |

`<audio>` を `decodeAudioData` に載せない。2分59秒ステレオはPCMで約60MBに展開され、iPhoneで詰む。
音量だけ gain を通すのは、**iOS Safari が media要素の `.volume` 代入を無視する**ため。

## 操作

| 入力 | 動作 |
|---|---|
| 寄 / A / ← | 間合いを詰める |
| 退 / D / → | 間合いを取る |
| 斬 / Space / J | 斬る |
| 受 / K / Shift（押しっぱなし） | 受ける |
| 崩 / L / ↓ | 崩す |
| 操作盤 ♪（上段中央） | ミュート（SE・BGM 共通） |

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
