# 実機資料から確定した事実

出典は Box の `02_取扱説明書 / Panasonic(KAIROS)`（社内共有フォルダー）。

- `Panasonic_Kairos_取扱ガイド_J.pdf`
- `Panasinc_KAIROS_AT-KC20M1G_仕様書.pdf`（KAIROS 全体の仕様書）
- `Panasonic_KAIROS_AT-KC10C1G_取扱説明書.pdf`
- `Panasonic_KAIROS AT-KC200_KC2000_取扱説明書.pdf`
- `Panasonic_Kairos Core Manager_操作説明書.pdf`
- `KAIROS_RestAPI_17_J.pdf`（REST API v1.7）

ここに書いたことは推測ではなく資料の記載です。**推測で入れていた仕様は下の「訂正」に集約**しました。

## 1. 構成

| 項目 | 内容 |
| --- | --- |
| メインフレーム | Kairos Core 200 **AT-KC200T / AT-KC200TL1** |
| コントロールパネル | Kairos Control **AT-KC10C1G**（標準・24XPT・1,140 mm）／ AT-KC10C2G（小型・12XPT・600 mm） |
| GUI | Kairos Creator **AT-SFC10G** |
| SDI 入出力 | 別売ボード **AT-KC20M1G**。1 枚 = 入力 8 / 出力 4 / REF IN・OUT、最大 4 枚 |
| 接続可能パネル | 合計 16 台（KC10C1G / KC10C2G 各最大 8 台） |
| オプション | RossTalk `AT-SFE01G` / NMOS `AT-SFE03G` / オーディオミキサー `AT-SF005G` / タッチパネル `AT-SFTC10G` |

Core 200 の主な数値：ST 2110 は 1.5G で 64 入力 / 40 出力、SDI は最大 32 入力 / 16 出力、
NDI HB **2 入力 / 2 出力**、SRT/RTSP/RTP/RTMP **8 入力 / 2 出力**、
マルチビューアー **4 出力・各最大 36 PiP**、クリッププレーヤー **2 CH**、オーディオプレーヤー 4 CH、
フレームディレイ 0〜12 フレーム、レイテンシー最小 1 フレーム、**6 色の独立タリー**、TSL 5.0。

> 「レイヤー数／**シーン（ME）数**／キーヤー数：機能制約なし、GPU 性能に依存、**GPU メーターで使用量を視認可能**」

シーン＝ME であることも、GPU メーターの存在も仕様書に明記されている。

## 2. コントロールパネル AT-KC10C1G の構造

取扱説明書「各部の名前とはたらき」より。

| ブロック | 内容 |
| --- | --- |
| シーンコントロールブロック（トップ / ボトム） | 各ブロックに **24 個 × 4 行**のボタン |
| トランジションコントロールブロック（トップ / ボトム） | 各ブロックに **6.6 型ワイド液晶パネル**とフェーダーレバー |
| マルチセレクトパネルブロック（トップ / ボトム） | 各ブロックに**表示内容が変わるボタン 12 個**。トップのみ USB |
| マルチファンクションブロック | **3 方向操作のジョイスティック 1 つ** |

取扱ガイドの「4 コントロールパネル」より：

- 行は**バスへのデリゲート**。初期レイヤー名は **BGD-A / BGD-B / Layer-1 / Layer-2**
- **Delegation ボタンを長押し**すると、そのバス内の他のレイヤーやシーンの Macros / Snapshots に選択を変更できる
- **シフトレベルは 1st〜4th**（「2nd シフトレベル」「4th シフトレベル」の記述あり）。24 XPT × 4 段 = 96
- **Profile 1〜8**（パネルプロファイル）。Panel Macro は各 Profile に属する
- **Smart Delegation**：シーン配置に応じて F1〜F7 が自動選択される（F8 が自動選択機能）
- 多目的キーパッドは **Menu / VTR / Macro / Snapshot** にデリゲートし、上下 2 つのディジポットで
  セクションとバンクを選ぶ
- **Sapphire モード**：Tally On-Air 以外のボタン色を消し、選択ボタンを青にする
- パネル単体ロック（0〜15 桁）、スクリーンセーバー
- **表示名に `#` を入れるとパネルの文字表示に強制改行**が入る
  （対象：Aux-Bus、Fx-Input、入力、シーン、シーンマクロ、スナップショット の Rename）
- フェーダーは **PGM/PST モードと A/B モード**に対応。シーン毎に切り替え
- Ver 1.8 以降、**1 台のパネルから最大 2 台の Kairos Core** を操作可能（Deck A / Deck B にデリゲート）

## 3. マクロ

- スコープは **Global Macros / Scene Macros / Panel Macros** の 3 種。
  Scene Macro はシーンに属し、Panel Macro は Profile 1〜8 に属する
- **記述言語は LUA**。Macro Edit ダイアログに「LUA 準拠のスタイルに自動 Format」「エラー出力の
  モニターボックス」「Insert Wait Template」「**Insert Function Template**」「Help Macro Description」
- Insert Function Template は **Action**（シーン制御、Global / Scene Macros、スナップショット呼出、
  Clip / Ram / Audio Player の **TMC**＝Tape Motion Control）と **Crosspoint** の 2 系統
- Crosspoint のパス表記の実物：**`Scenes / Main / Layers / Background / SourceA`**
- Wait は **Time（msec / sec）** または **Frame**。ユーザー待ちは **`{wait_user()}`**
  （到達で停止、再実行で続行）
- **シーンのタリーで自動実行**：OnAir（Red）で **`AUTOSTART`**、OffAir で **`AUTOSTOP`** という名前の
  シーンマクロが自動実行される
- Apps も LUA。2 本まで同時実行、Production ファイルに保存される
- 標準 Production には「Main」シーンと「**BgdMix**」トランジションが含まれる

## 4. シーンとレイヤー（原文）

> シーンは、一般的なスイッチャーでの ME に相当します。

> Layer ＝ レイヤーは合成を目的とする一般的なキーヤーのように扱われますが、1 つのレイヤーに
> 属性（輝度キーおよびクロマキー、マスク、Wipe、DVE、カラー補正など、同じタイプのものでも）を
> 複数組み合わせて処理できる点が、キーヤーとは異っています

> Background バスのような裏表のあるバスの操作は、シーン毎に PGM/PST（フリップフロップ）と
> A/B モードを切り替え

Layers パネルは「レイヤー名およびレイヤーの順序と、各バスの色」「各バスに設定されたソース」
「バスごとの On/Off 状態」を表示する。

## 5. Kairos Creator のメニュー構造

> Kairos Creator のメニューは、各１階層以内になるよう設計された、革新的な新しいコンセプトに
> 基づいています。その結果、ユーザーは別のメニューページへ、メニュータブを最大 2 回クリック
> するだけで移動することができます。

- 起動時は **Mixer タブの Control サブメニュー**。Control ページは「タッチ操作可能な Control Panel」
- Mixer 配下：Control / Scenes / Transitions / Snapshots / Audio Mixer / User Management
- Config 配下：Inputs / Panel / Triggers と Listeners
- Setup 配下：System Settings / Input Settings / Output Settings / System Info
- ほかに Macros（Control / Edit / Apps）、Multiviewer、Scenes
- GUI 右上に **QSFP 帯域幅・CPU・GPU** のメーター
- Ctrl+Tab でタブ循環、Undo（Ctrl+Z）/ Redo（Ctrl+Y）
- REST API の分類：AUX-All / AUX-Delegation / Inputs / Macros / Multiviewer / Scenes

## 5.5 REST API（v1.7）

`KAIROS_RestAPI_17_J.pdf` より。**このアプリの出力はここに合わせている。**

### 接続

```
http(s)://<ip>(:<port>)/<json-pointer>
IP アドレス（工場出荷時）: 192.168.10.10
ポート番号（工場出荷時）: 1234
```

- 末尾の `/` は不可（`404 Invalid url`）
- 認証は **Basic / ダイジェスト**。ユーザー名は **`Kairos`**。
  パスワードは v1.3.2 以降の初回起動時に設定したもの
- **GET と PATCH のみ**。PUT / DELETE / POST は `400 Bad Request`
- PATCH は **`application/merge-patch+json`**（RFC7396）
- 応答コードは `200` / `400` / `404`

```
$ curl http://Kairos:password@192.168.10.10:1234/inputs/IP1
$ curl --request PATCH http://Kairos:password@192.168.10.10:1234/scenes/Main/Background -d "{\"sourceA\":\"IP1\"}"
```

### エンドポイント

| 対象 | パス | メソッド |
| --- | --- | --- |
| Input | `/inputs` `/inputs/<input>` | GET |
| Macro | `/macros` `/macros/<macro>` | GET / PATCH |
| Scene Macro | `/scenes/<scene>/macros/<macro>` | PATCH |
| Aux | `/aux` `/aux/<aux>` | GET / PATCH |
| Multiviewer | `/multiviewers` `/multiviewers/<0-1>` `/multiviewers/<n>/sdp` | GET / PATCH |
| Scene | `/scenes` `/scenes/<scene>` | GET |
| Layer | `/scenes/<scene>/<layer>` | PATCH |
| Action | `/scenes/<scene>/actions/<uuid>` | PATCH |
| Snapshot | `/scenes/<scene>/snapshots/<snapshot>` | PATCH |

`<uuid>` でも指定できる。

### リクエストボディ

| 対象 | ボディ |
| --- | --- |
| Layer | `{"sourceA": "Black"}` / `{"sourceB": ...}` |
| Action | `{"state": "play"}` |
| Macro | `{"state": "play"}` |
| Snapshot | `{"state": "recall"}` |
| Aux | `{"source": "Black"}` |
| Multiviewer | `{"preset": 11}` |

ソースは各要素の **`sources` リストに載っている値だけ**が受け付けられる。
載っていない値や余分なキーを含めると `400 Bad Request`。

### オブジェクトのパラメーター

| 型 | パラメーター |
| --- | --- |
| Base | `uuid`（R） |
| Input | `index` `name` `tally`（すべて R） |
| Layer | `name`（R）`sourceA`（R/W）`sourceB`（R/W）`sources`（R） |
| Aux | `index` `name` `source`（R/W）`sources` |
| Macro | `color` `name` `path` `state`（W） |
| Snapshot | `name` `state`（W） |
| Multiviewer | `index` `name` `preset`（W）`presets` `sdp` |
| Scene | `actions` `layers` `macros` `name` `path` `snapshots` `tally` `uuid` |

**Layer に opacity や ON/OFF のパラメーターはない。** レイヤーの表示・非表示は
`actions` 経由で行う（下記）。

### actions（v1.7 で GET Scenes の応答に追加）

`GET /scenes` の応答にある `actions` の実例（原文）:

```
{"name": "Main:auto",                "uuid": "a9564d77-..."}
{"name": "Main:cut",                 "uuid": "9ee04835-..."}
{"name": "Background:show_layer",    "uuid": "3702f380-..."}
{"name": "Background:hide_layer",    "uuid": "cd708c2b-..."}
{"name": "Background:toggle_layer",  "uuid": "68f42656-..."}
{"name": "Layer-1:show_layer",       "uuid": "b963f817-..."}
{"name": "BgdMix:transition_cut",    "uuid": "4943ecfe-..."}
{"name": "BgdMix:transition_auto",   "uuid": "4817304a-..."}
{"name": "L1:transition_auto",       "uuid": "0502ed80-..."}
{"name": "M1- Main:play",            "uuid": "1df3f352-..."}
{"name": "SNP1Main:recall",          "uuid": "2ed9cca6-..."}
```

→ **レイヤー ON/OFF、AUTO / CUT、マクロ実行、スナップショット呼出はすべて action**。
名前は `<対象>:<動作>` 形式で、実行は uuid を指定して `{"state":"play"}` を PATCH する。
アプリ側は起動時に `GET /scenes` で名前 → uuid を解決する。

### ソースの実名（応答例より）

`IP1` `IP2` … / `CP1` `CP2`（クリッププレーヤー）/ `RAM2` `RAM3`（RAM プレーヤー）/
`IS1`（イメージストア）/ `ColC`（カラーマット）/ `ColorBar` / `Black`

シーンのレイヤー名は `Background` `Layer-1` `Layer-2` `Layer-3` …、
トランジション名は `BgdMix` `L1` `L2` …。

### REST では実行できないもの

`WAIT` / `WAIT USER` / `GPO` / TMC（Clip・RAM・Audio Player の再生制御）は
エンドポイントがない。**これらはマクロ側で実行する**。
アプリのブロック一覧では `MACRO` バッジを付けて区別している。

## 6. 訂正（前回まで推測で入れていたもの）

| 前回 | 実際 |
| --- | --- |
| SHIFT は「通常 / SHIFT」の 2 段 | **1st〜4th の 4 段**。24 × 4 = 96 |
| パネル表示名は 8 文字、UMD は 16 文字（TSL 3.1 由来） | KAIROS は **TSL 5.0**。文字数の記載は資料になし。実際の仕組みは **`#` による強制改行** |
| パネルの行は PGM / PST / KEY / AUX SEL の固定 | 行は**バスへのデリゲート**（BGD-A / BGD-B / Layer-1 / Layer-2）で、押すたびに変えられる |
| マクロは独自のコマンド列 | **LUA スクリプト**。パスは `Scenes/Main/Layers/Background/SourceA` 形式 |
| 入力の KAIROS 名を `IP01` などとしていた | 実名は **`IP1` `IP2`** … / `CP1` / `RAM2` / `IS1` / `ColorBar` / `Black` |
| レイヤーの ON/OFF をレイヤーのパラメーターと想定 | Layer は `sourceA` `sourceB` `sources` のみ。ON/OFF は **action** |
| レイヤーは KEY / DVE の固定種別 | 1 レイヤーに**複数の属性を組み合わせる**（輝度キー＋クロマキー＋マスク＋DVE …） |
| キーは「リニアキー」 | KAIROS の列挙は**ルミナンスキー / クロマキー**。リニアの表記はない |
| マクロは 1 種類 | **Global / Scene / Panel** の 3 スコープ |

## 7. まだ確認が必要なこと

1. ~~REST API のコマンド表~~ → **入手済み。5.5 章のとおり実装した**
2. **LUA の関数名**。パス表記は確定したが、`crosspoint()` `transition_auto()` などの関数名は
   Creator の「Insert Function Template」「Help Macro Description」で確定させる必要がある。
   ただし **REST で必要な操作はすべて賄えるため、優先度は下がった**
3. `state` に指定できる値の全一覧（マニュアルには `play` `recall` の例のみ。`stop` は
   action 名からの推定）
4. `tally` の値の意味（`GET /inputs` は 0 / 1、`GET /scenes` は 3 という例。定義の記載なし）
5. パネル表示名の最大文字数（`#` 改行の挙動含む）
6. 当該スタジオの Core 200 の SDI ボード枚数と実入出力数
7. Deck A / Deck B に何を割り当てて運用するか（デュアル Core を使うかどうか）
