# 設計メモ — KAIROS SUPPORT

対象機材: **KAIROS Core 200** ／ **大型コントロールパネル**

実機仕様の根拠は [kairos-facts.md](kairos-facts.md)（メーカー資料から確定）、
表記ルールの根拠は [ui-research.md](ui-research.md)（他社スイッチャー UI 調査）にある。

## 1. 解こうとしていること

- レイヤー合成が自由すぎて、どこを触ると本線が変わるのか分かりにくい
- マクロがコマンド列で、書ける人と書けない人が分かれる
- **パネルの仕込みが分かりづらい**。特に SHIFT の裏の割当が見えない
- 入力・マルチビュー・パネル・音声が別々の設定に散っている

ATEM が広く使われている理由は機能の多さではなく、
**「押せば画が変わる」が最初の 5 分で分かること**にあります。
そこを KAIROS に持ち込むのが狙いです。

## 2. UI の骨格

KAIROS Creator は「メニューは各 1 階層以内、最大 2 クリックで全ページへ」という
構造を持つ。これに合わせてメインタブ＋サブタブの 2 階層にした。

```
┌ 上部バー：識別 / メインタブ / 接続先 / TC / ON AIR ───────┐
├ サブタブ ＋ QSFP・CPU・GPU メーター ──────────────────────┤
├ ワークスペース（ドッキングされたペイン、1px 罫線で区切る）─┤
├ コントロールサーフェス（常時表示：PGM・PVW・BANK・T バー）┤
└ ステータスバー：メッセージ / 未書込 / 選択 / 各種カウント ┘
```

- カードを浮かべない。ペインは画面端まで詰め、境界は 1px の罫線だけ
- 各ペインが個別にスクロールする（ページ全体は動かさない）
- 画面はスクロールする文書ではなく、**一画面で完結する道具**

## 3. UI の原則

作る側の判断がぶれないように明文化しておきます。番号の後ろは根拠。

1. **英字は大文字の略号。** `PGM` `PST` `AUX` `KEY` `DSK` `XPT` `FTB` `BKGD`。
   例外は単位（dB, ms, F）、製品名、生成コマンド。
   — Panasonic AV-HS400A のパネル刻印、ATEM のボタン表記に合わせる
2. **バスは PGM / PST。** PVW は出力の名前としてだけ使う。
   — Panasonic・Roland・FOR-A の日本語マニュアルの表記
3. **アプリは自分を説明しない。** ヒント枠・説明文・注釈アイコンを置かない。
   ラベルは名詞、操作は短い動詞。
   — データインク比、情報アクセスコストの最小化
4. **カードにしない。** ペイン＋罫線。角丸と影とグロウは使わない
   （タリーの色だけは例外）。
5. **密度を上げる。** 行の高さ 26〜28px、本文 12.5px、ラベル 10.5px。
6. **アクセントは一色だけ、意味のある場所だけ。**
   アンバー ＝ 選択・編集中・未書込。赤 ＝ PGM、緑 ＝ PST。
7. **状態は必ず同じ場所に出す。** 未書込・アラーム・GPU 使用率・空きキー・重複。
   — 一目で状況が分かること（situational awareness）
8. **時間はフレームで持つ。** `30 F`。秒は使わない。
   — ATEM の RATE、Panasonic のパネル表記
9. **データはきれいすぎないほうがいい。** 未結線、フォーマット不一致、帯域警告、
   名前の文字数超過。現場に実在するものを最初から入れておく。

## 4. 配色・書体

| 役割 | 値 |
| --- | --- |
| 地 / ペイン / ヘッダー | `#0E1114` / `#13171B` / `#1A1F25` |
| 罫線 | `#232930`（通常）`#333B44`（強） |
| 文字 | `#DCE2E8` / `#8D96A1` / `#5C646E` |
| アクセント | アンバー `#E8A33D` |
| 意味色 | PGM `#E23B4E` / PVW `#2FBF6B` / IP・キー `#5A8FC7` / 警告 `#D9822B` |
| 書体 | IBM Plex Sans ＋ IBM Plex Sans JP、数値とコマンドは IBM Plex Mono |

IBM Plex は計測機器・エンジニアリング寄りの表情で、副調の道具に合います。
モニターのサムネイルには実行時生成のグレインを重ね、CSS のグラデーション板に
見えないようにしています。

ダーク一択。副調のモニター前で使う道具なので、明るい面を作らないこと自体が要件です。

## 5. KAIROS の概念 ↔ このアプリでの見せ方

| KAIROS | 見せ方 |
| --- | --- |
| Mixer / Macros / Multiviewer / Config / Setup | そのままメインタブにする |
| Mixer > Control / Audio Mixer、Config > Inputs / Panel | サブタブにする |
| Scene | シーンのタブ列。M/E に相当する単位として扱う |
| Layer | レイヤースタック。上が前面。1 レイヤーに複数の属性（輝度キー・クロマキー・マスク・DVE・カラー補正）を組み合わせる |
| Background | 各シーンの固定名レイヤー。名前は変えない |
| Background の SourceA / SourceB | PGM / PST バス。シーン毎に PGM/PST と A/B を切り替え |
| Clip Player / RAM Recorder | 入力の種別としてそのまま表示（`CLIP PLAYER` `RAM REC`） |
| GPU 負荷 | ステータスバーに常時表示 |
| Layer トランジション | NEXT（BKGD / KEY1 / KEY2）＋ T バー ＋ AUTO / CUT |
| Snapshot | 一覧から呼出。マクロからも呼べる |
| AUX | スイッチャー画面に 12 系統を常置。行き先とソースを 1 行で |
| Macro | ブロックエディタ。並べた結果をコマンドに変換 |

## 6. マクロエディタ

KAIROS のマクロは **LUA スクリプト**で、スコープは **Global / Scene / Panel** の 3 種。
Creator の Macro Edit には「Insert Function Template」（Action と Crosspoint）があり、
Crosspoint のパス表記は `Scenes / Main / Layers / Background / SourceA`。

ブロックはこの Insert Function Template に 1:1 で対応させています。

| ブロック | 生成される LUA（サンプル） |
| --- | --- |
| CROSSPOINT | `crosspoint("Scenes/Main/Layers/Background/SourceB", "Inputs/IP02")` |
| AUX | `crosspoint("Aux/AUX3", "Scenes/Main/Program")` |
| AUTO / CUT | `transition_auto("Scenes/Main/Transitions/BgdMix", 30)` |
| LAYER ON / OFF | `layer_enable("Scenes/Main/Layers/Layer-1", true)` |
| SNAPSHOT | `snapshot_recall("Scenes/Main/Snapshots/SNAP1")` |
| MACRO | `macro_play("Macros/Global/M09")` |
| TMC | `tmc("Players/RamPlayer1", "Play")` |
| MULTIVIEWER | `multiviewer_preset("Multiviewer/MV1/Presets/P2")` |
| GPO | `gpo_send(2)` |
| WAIT | `wait_frame(60)` / `wait_msec(500)` |
| WAIT USER | `wait_user()` |

シーンのタリーで自動実行される **`AUTOSTART` / `AUTOSTOP`** もマクロ一覧に出しています。

**生成コードは隠さない。** 隠すと「何をされたか分からない道具」になり、現場では信用されません。
書けない人はブロックだけ見て、書ける人は右の LUA 欄で確認する二層構造にしています。

> パス表記は資料で確定しましたが、**関数名はまだ仮**です。Creator の
> 「Insert Function Template」「Help Macro Description」で確定させます。

## 7. パネル画面（AT-KC10C1G）

実機の構造をそのまま描いています。

```
DECK A / TOP      [DELEG][24 XPT × 4 行]  [6.6型液晶 / F1-F8 / AUTO CUT / フェーダー]   [KEYPAD 12 + ディジポット]
DECK B / BOTTOM   [DELEG][24 XPT × 4 行]  [6.6型液晶 / F1-F8 / AUTO CUT / フェーダー]   [KEYPAD 12 + ディジポット]
                                                                                        [ジョイスティック]
```

- 行は**バスへのデリゲート**（初期レイヤー名 BGD-A / BGD-B / Layer-1 / Layer-2）
- ソースの並びは**シフトレベル 1st〜4th** でページングされる（24 × 4 = 96 スロット）
- Profile 1〜8、Sapphire モード

仕込みで効くのは次の 5 つです。

1. 実機と同じ配置で、押す前に何が起きるか分かる
2. 割当一覧で **1st〜4th を横並び**に見る（純正で見えないのはここ）
3. XPT を選ぶと**その列の 4 段すべての割当**が右に出る
4. **重複割当の警告** — 同じソースが複数スロットに入っているのは事故のもと
5. パネル表示名を `#` 付きで編集すると、**キーの見え方（改行）がその場で分かる**

## 8. 次に決めること

1. **接続方式** — Core 200 への制御は何を使うか（制御ポート / REST）
2. **書き込み単位** — 差分書き込みが可能か、プロジェクト全体の入れ替えか
3. **同時操作** — KAIROS Creator や実パネルと同時に触ったときの整合
4. **パネル型番** — 実機のキー数・行構成をモックに反映する
5. **オフライン編集** — 実機なしで作って現場で流し込む運用を前提にするか
