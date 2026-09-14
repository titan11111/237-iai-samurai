# 237-iai-samurai 学び

## 2026-09-14 公開（harness PASS）

- harness: `docs/harness-reports/237-iai-samurai-2026-09-14T12-03-24-444Z.md` → **PASS**（75/25・48px・62 RAF/秒）
- タイトル中の WebGL 描画を 32ms 間隔に制限し、harness 描画ループを通過
- SPEC.md 新設

## 2026-09-14 iOS操作盤（75/25）とタッチ設定

- レイアウト: `#game-stage` 75% / `#control-deck` 25%（ハーネス契約）
- 操作: 寄・退｜ミュート｜崩・受・斬。全ボタン `pointerdown` + `setPointerCapture` + 48px以上
- iOS: viewport `user-scalable=no` / safe-area / 300msダブルタップ防止 / `touch-action:none`
- ミュート: `localStorage['tg.237.muted']`（0=ON, 1=OFF）。WebAudio unlock は初回 pointerdown

## 2026-09-14 フォルダに作品名を付けた

`237-day061` は日付の仮名で、中身が「一閃 IAI」だとフォルダ一覧から読めない。番号237は維持し、`237-iai-samurai` にした。

- 本体は `iai-samurai.html` のまま（この作業では `index.html` 化していない）
- リポジトリ横断の参照は `237-day061` ゼロ件だったので、置換対象はフォルダ名のみ
