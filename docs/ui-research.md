# 他社スイッチャー UI 調査メモ

> **注記**：本書は他社製品の一般的な作法の調査。KAIROS 自体の仕様は
> [kairos-facts.md](kairos-facts.md)（実機資料から確定）が優先する。
> 特に 3 章の文字数制限は KAIROS には当てはまらない（KAIROS は TSL 5.0、
> パネル表示名は `#` による強制改行）。

このアプリの表記ルールを決めるために、実機ソフトと物理パネルの表記を調べた記録。
「なんとなく良さそう」で決めた項目をなくすことが目的。

> 調査環境の制約でメーカー PDF への直接アクセスができず、検索結果の要約を根拠にしている。
> ★印は実装前に実機・実マニュアルで裏取りしたい項目。

## 1. 英字は大文字の略号

| 出典 | 内容 |
| --- | --- |
| Panasonic AV-HS400A 操作説明書のパネル刻印 | `KEY` `DSK` `PinP` `AUX` `CLN` `PVW` `PGM` `USER` `F1〜F5` |
| Blackmagic ATEM マニュアル | `BKGD` `KEY 1〜4` `TIE` `FTB` `RATE` `AUTO` `CUT` |
| Sony ICP-X7000 / MVS | パネルは略号ニーモニック中心。ボタン配置は状態認識の速さを優先 |

→ **UI の英字はすべて大文字の略号**にする。例外は単位（dB, ms, Mbps, GB, F）、
製品名（KAIROS Core 200）、生成コマンド（コードはコードとして小文字のまま）。

## 2. PGM / PST（フリップフロップ）

Panasonic・Roland・FOR-A の日本語マニュアルはいずれも
「PGM/A バス」「PST/B バス」と書き、**PVW は出力の名前**として使う。
FOR-A も PGM/PST 行でクロスポイント操作と説明している。

→ コントロールサーフェスの 2 段目は **PST**。PVW はモニターの表示にだけ使う。

## 3. 名前の文字数制限が「仕込みの難しさ」の正体

| 出典 | 制限 |
| --- | --- |
| Grass Valley Karrera（OLED Name） | パネル表示名は **8 文字** |
| TSL UMD Protocol v3.1 | マルチビューのラベルは **16 文字**（16 文字すべてを送る） |
| Ross Carbonite | ソース名と TSL ID を入力ごとに設定 |

現場で名前が崩れるのは、長い和名を 8 文字と 16 文字へ詰める作業が
別々の画面に散っているから。

→ 入力一覧に**パネル表示名の列**を常設する。
→ パネル画面のキーには表示名をそのまま出す（実機と同じ見え方になる）。

**※ KAIROS では上の 8 / 16 文字は当てはまらない。** KAIROS は TSL 5.0 対応で、
パネル表示名は `#` を入れて強制改行させる仕組み。詳細は
[kairos-facts.md](kairos-facts.md) を参照。

## 4. KAIROS 自身の語彙に合わせる

| 出典 | 内容 |
| --- | --- |
| KAIROS 操作マニュアル | GUI は `Mixer`（Control / Scenes / Transitions / Panel）、`Multiviewer`、`Macro`、`System`。「メニューは 1 階層、最大 2 クリックで全ページへ」 |
| Vizrt Viz Mosart の KAIROS 連携ドキュメント | 各シーンには **Background という固定名のレイヤー**が必要。PP（Program/Preview）領域は必須 |
| Panasonic のマクロ機能説明 | マクロでできること＝**クロスポイント選択 / ソース切替 / AUTO トランジション / AUX 再アサイン / MultiView レイアウト変更 / RAM レコーダー・クリッププレーヤーの再生** |
| KAIROS Creator の画面説明 | アイコントレイに **Main Macro 録画ボタンと GPU 負荷メーター** |

→ タブを `MIXER / INPUTS / MULTIVIEW / MACRO / PANEL / AUDIO / SYSTEM` に。
→ レイヤー名は `BKGD` ではなく **Background**。
→ マクロのブロックは上の実機リストに 1:1 で対応させる
　（`XPT` `AUTO TRANS` `KEY` `SCENE` `SNAPSHOT` `AUX` `MULTIVIEW` `CLIP PLAYER` `RAM REC` …）。
→ 入力の種別も KAIROS のオブジェクト名（`CLIP PLAYER` `RAM REC` `STILL`）に。
→ ステータスバーに **GPU 使用率**を出す。KAIROS 使いには意味のある数字。

## 5. 制御方式

| 出典 | 内容 |
| --- | --- |
| Panasonic / 業界紙 | ver 1.7 で **RESTful API** とクライアント・サーバー方式（複数 GUI から 1 台の Core を制御） |
| Panasonic ニュース（2022) | ver 1.3.2 で **RossTalk**（AT-SFE01 アクティベーションキー） |
| 同上 | ver 1.7 以降 **Raw Panel**（SKAARHOJ パネル対応） |
| Viz Mosart 連携 | 簡易制御プロトコルは 5 秒ごとの keep-alive |

→ SYSTEM 画面は「接続テスト」ではなく、**実在する 3 つの経路**を並べて状態を出す。★

## 6. 時間はフレームで持つ

ATEM の RATE はフレーム数。Panasonic のパネルも同様。

→ トランジション・キー・WAIT はすべて **フレーム表記（`30 F`）**。秒は使わない。

## 7. マクロはスロットで数える

ATEM は 100 スロット・20 件ずつ表示。名前と説明を持ち、収録中は赤枠、
`add pause` ボタンで停止を差し込む。

→ マクロは番号スロット（M01…）で並べ、未作成も一覧に出す。
→ 「操作待ち（PAUSE）」をブロックとして持つ。

## 8. 画面の作り方

- 情報アクセスコストの最小化：視線移動そのものがコスト。関連する情報は隣に置く
- データインク比：同じ情報量ならインクは少ないほうがよい
- 一目で状況が分かること（situational awareness）が副調の設計原則

→ 説明文・ヒント枠を置かない。ラベルは名詞、操作は短い動詞。
→ 状態（未書込・アラーム・GPU）は常に同じ場所に出す。

## 出典

- [ATEM Software Control（Blackmagic Design）](https://www.blackmagicdesign.com/products/atemmini/software)
- [ATEM Mini マニュアル（Switching / Macros / Keying）](https://usermanual.com/support/blackmagicdesign/web-manual/atem-mini/atem-software-control)
- [Panasonic AV-HS400A 操作説明書](https://www.arkvideo.co.jp/_sys/wp-content/uploads/AV-HS400A_OperationManual.pdf)
- [Panasonic AV-HS410N 取扱説明書（操作・設定編）](https://eww.pass.panasonic.co.jp/pro-av/support/dload/hs410_ver201/operation_ope_j.pdf)
- [Panasonic KAIROS 操作マニュアル（ManualsLib）](https://www.manualslib.com/manual/2314208/Panasonic-Kairos.html)
- [KAIROS Creator Software（Panasonic）](https://connect.na.panasonic.com/av/video/kairos/creator-software)
- [Viz Mosart — Panasonic Kairos Vision Mixer](https://docs.vizrt.com/viz-mosart-admin-guide/5.11/Panasonic_Kairos_Vision_Mixer.html)
- [Panasonic Connect releases KAIROS 1.3.2 with RossTalk](https://na.panasonic.com/news/panasonic-connect-releases-kairos-version-1.3.2-software-upgrade-with-rosstalk-protocol)
- [TSL UMD Protocol](https://tslproducts.com/wp-content/uploads/TSL-UMD-protocol.pdf)
- [Ross Carbonite Black セットアップ（Source Names / TSL ID）](https://www.manualslib.com/manual/2626345/Ross-Carbonite-Black.html?page=21)
- [Ross DashBoard / Carbonite](https://help.rossvideo.com/ultrix-carbonite/Topics/Features/Features.html)
- [Grass Valley Karrera の OLED Name 8 文字（TD Guide）](https://s3.us-east-2.amazonaws.com/sidearm.nextgen.sites/tamu.sidearmsports.com/documents/2020/9/17/12MP_TD_Training_Booklet.pdf)
- [vMix User Interface Overview](https://www.vmix.com/help24/UserInterfaceOverview.html)
- [Sony XVS / MVS シリーズ ブローシャー](https://assets.pro.sony.eu/Web/supportcontent/XVS-MVS-Series-Brochure-Feb-2018.pdf)
- [FOR-A ビデオスイッチャーの特長](https://www.for-a.co.jp/products/articles/tech_sw_02/)
- [Bitfocus Companion ユーザーガイド](https://companion.free/)
- [The Basic Principles Of Production Control Room Design](https://www.thebroadcastbridge.com/content/entry/18982/video-control-rooms-designing-the-heart-of-your-production-studio)
- [Control Room Layout and Design Principles](https://www.activu.com/control-room-layout-and-design-principles-for-mission-critical-environments/)
