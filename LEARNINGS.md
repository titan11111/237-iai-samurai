# 237-iai-samurai 学び

## 2026-09-16 BGM（Steel_in_the_Grass.mp3）を鳴らした

**やったこと**

- `<audio loop preload="none">` ＋ WebAudio の gain 経由で再生。最初のタップで開始、♪ボタンで停止／再開、裏へ回ったら停止
- 併せて**押せなかったミュートボタンを押せるように直した**（下記）

**詰まった点と原因**

- **harness が `page.goto` の20秒で落ちた**。原因は `preload="auto"`。Chromiumはメディアを絞って流すので4.3MBの取得が20秒を超えて続き、**`networkidle` が永久に来ない**。`preload="none"` にして解決（起動時の通信量も 0.06MB のまま）
- `decodeAudioData` は使わない。179秒ステレオ44.1kHzはPCMで約60MB。`<audio>` のまま流せば数MBで済む
- **iOS Safari は `audio.volume` への代入を無視する**。音量を握る唯一の手段が `createMediaElementSource` → `GainNode`。フェードもgainで書く
- ミュートで `pause()` した瞬間、Chromiumが進行中のストリームを切って `net::ERR_ABORTED` を出すことがある。harness は requestfailed を1件でも FAIL にするので、`preload="none"`（タップまで通信しない）が保険にもなっている

**HE-AAC化（同日追加）**

- `afconvert -f m4af -d aach -b 64000 -s 3` で **4.11MB → 1.43MB（65%減）**。長さ179.5秒は保持
- **ffmpegの内蔵aacエンコーダではHE-AACを出せない**（`-profile:a aac_he` は libfdk_aac が要る）。macOSなら CoreAudio の `afconvert` が標準で持っている
- `<source>` を m4a → mp3 の順に置いた。実測で**対応ブラウザは m4a だけを取得し、mp3 は1バイトも落とさない**。フォルダは5.6MBだが通信は1.4MB
- WebKit（iOS Safariと同じエンジン）で再生を実測: `canPlayType('audio/mp4; codecs="mp4a.40.5"')` = probably、再生時間が進むことを確認。mp3は "maybe" 止まりで、**WebKitはm4aを選ぶ**

**見つけた既存不具合：ミュートが押せなかった**

実測（Playwright・要素の矩形を直接測定）:

| 画面幅 | 改修前の重なり | 修正後 |
|---|---|---|
| 375×667 | `snd×bB 47×48px` `snd×bG 5×48px` `bR×bB 7×56px` | 重なり0 |
| 390×844 | `snd×bB 48×48px` `snd×bG 3×48px` `bR×bB 6×59px` | 重なり0 |
| 430×932 | `snd×bB 59×48px`（**ミュートが完全に隠れていた**） | 重なり0 |

- 原因は操作盤の横幅オーバー。5つの操作ボタン＋ミュートは 390px 幅に並ばず、flexが縮みきって内容がはみ出し、重なっていた
- 直し方: ①操作列を操作盤の**下端**へ寄せ、空いた上段中央へミュートを逃がす ②`.mv` `.at` `#bA` の寸法を 17→16vw / 18→16.5vw / 21→19.5vw へ詰める
- 全ボタン48px以上は維持（最小60px）。harness の該当項目も PASS
- **教訓**: harness は「48px以上」「操作盤内に収まる」は見るが、**ボタン同士の重なりは見ない**。音を足すまで誰も気づかなかった

## 2026-09-16 顔と応援者を足した

**やったこと**

- 侍の顔（眉・白目・瞳・鼻筋・口・顎当て）＋鉢巻を実装。旧 `men`（面）は口を塞いでいたので顎へ落とした
- 輪の外に応援者5人（立3＝扇2・幟1／正座2＝置き提灯）。決着で歓声が高ぶる `hype` を共有

**分かったこと**

- 顔の部品は球面の極座標で置くと速い。`onFace(w,h,d,mat,yaw,pitch,r,roll)` で置き、`lookAt(position*3)` で法線方向へ向ける。オイラー角を手で合わせる必要がない
- **侍は常に横顔でしか見えない**（`root.rotation.y=±π/2`）。正面向きに作り込んでも画面には出ない。効くのは鼻筋・顎の輪郭・鉢巻の色といった**シルエット**
- 鉢巻は最初 y=1.872 に置いたら眉と目を覆った。額は思ったより高い（髪の下端は頭中心+.033）。y=1.902 へ上げて解決
- **縦持ちの画角は横に狭い**。fov40・アスペクト0.687だと視野の半幅は奥行の約0.25倍しかない。半径7.4の弧に等間隔で5人置くと両端（x=±7）は画面外だった。奥側だけに寄せて `|x| ≤ 4.5` に収めた
- 提灯は手に持たせると正座の人では地面すれすれで見えない。**脇に立てる置き提灯**にしたら画面で光として効いた
- 幟の板は `rotation.y=π/2` を入れていたせいで真横を向き、線にしか見えなかった。応援者は中央（＝カメラ側）を向いているので回転は不要

**検証（2026-09-16 時点）**

- Playwright(Chromium・390×844・DPR2)で描画を実測。`pageerror` 0件・コンソールエラー0件
- 顔／応援者ともスクリーンショットで実物を確認（タイトル・対戦・決着・寄り）
- harness は**マシン高負荷のため描画ループ判定が信用できない状態**。詳細は下の節へ


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

## 2026-09-19 旧エントリURLの404を修復
- 症状: `https://titan11111.github.io/237-iai-samurai/iai-samurai.html` が **404**。本体（`/237-iai-samurai/`）は 200 で生きていた
- 原因: エントリを `iai-samurai.html` → `index.html` へ改名したため、**改名前に配ったリンクだけが死んだ**
- 対処: `iai-samurai.html` を index.html へのリダイレクト専用ページとして復活（meta refresh ＋ `location.replace()`。`?query`・`#hash` も引き継ぐ）
- 検出元: `_tools/check-legacy-entry.sh`（245の同種事故を機に新設）。本番URLへcurlを撃って検出
- 鉄則8: エントリ名を変えたら旧名をリダイレクトで必ず残す（本体URLが200のままなので気づけない）

### 【訂正】上の「404だった」は誤り（2026-09-19 同日中に判明）
- gitで裏を取った結果、`iai-samurai.html` は**このリポジトリで一度も公開されていなかった**（`git cat-file -e <修復コミット>^:iai-samurai.html` → 不在）。
  改名はローカルフォルダ内で完結しており、リポジトリは改名**後**に作成されている
- つまり `…/237-iai-samurai/iai-samurai.html` というURLは**元から存在しない**。「配ったリンクが死んだ」という上の記述は**誤り**
- 置いたリダイレクトは**害はないが、壊れていたものを直したわけではない**（将来その名前で来た人を受けるだけの保険）
- 誤認の原因: LEARNINGS.md の本文を証拠として扱ったこと。**本文は作業メモであって証拠ではない。証拠はgit履歴**
- 検出器も v2 で「git履歴に存在 かつ HEADに不在」判定へ作り直した（`_tools/check-legacy-entry.sh`）
