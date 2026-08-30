# KAIROS SUPPORT

Panasonic **KAIROS Core 200** ＋ **大型コントロールパネル**の設定を、
現場の人が触れる形で行うためのサポートアプリ。

KAIROS はレイヤー合成・IP 入出力・マクロと自由度が高い一方、
設定がスクリプト的で、特に**パネルの仕込み**が分かりづらい。
このアプリはその間に立って、画面で組み立てた操作を KAIROS のコマンドに翻訳します。

## 対象システム

| 項目 | 内容 |
| --- | --- |
| メインフレーム | KAIROS Core 200（AT-KC200T） |
| SDI | 32 IN / 16 OUT（AT-KC20M1G 4 枚フル実装） |
| コントロールパネル | AT-KC10C1G（24 XPT） |
| Core | 1 台（デュアル Core は使わない） |

## 現在の状態

クリック可能な HTML モックアップ（単一ファイル・ライブラリ依存なし）。
実機とは通信しません。すべて画面上のダミー状態です。

```
mockup/index.html    ブラウザで開くだけで動く
docs/concept.md      画面構成・設計方針・UI の原則
docs/kairos-facts.md 実機資料（取扱ガイド・仕様書）から確定した事実と訂正
docs/ui-research.md  他社スイッチャー UI の調査と、そこから決めた表記ルール
```

## 画面

タブ構成は KAIROS Creator に合わせています（メインタブ＋サブタブ、最大 2 クリック）。

| メイン | サブ | 内容 |
| --- | --- | --- |
| MIXER | CONTROL | PGM / PST、シーン、レイヤー、AUX 12 系統、SNAPSHOT、収録・配信・GPU・警告 |
| MIXER | AUDIO MIXER | フェーダー、AFV 対応、出力ルーティング |
| MACROS | CONTROL | ブロックを並べて組み、**REST リクエスト / LUA を自動生成**。Global / Scene / Panel |
| MULTIVIEWER | LAYOUT | レイアウトとペイン割り当て。タリー枠は PGM / PST と連動 |
| CONFIG | INPUTS | 入力一覧。パネル表示名（`#` で改行）を編集 |
| CONFIG | OUTPUT | **SDI 出力 16 本**の割り当て（ボード B1-B4 単位）と I/O の使用状況 |
| CONFIG | **PANEL** | **AT-KC10C1G を実機どおり再現。シフト 1st〜4th を並べて表示** |
| SETUP | SYSTEM | 接続、制御プロトコル、差分書き込み、テンプレート |

画面下の**コントロールサーフェス**（PGM / PVW バス、BANK、T バー、AUTO / CUT）は
どの画面でも常時表示。設定中も本線を見失いません。

## パネル画面について

純正ツールでいちばん分かりづらいのは「**シフトの裏に何が入っているか**」です。
AT-KC10C1G は 24 XPT × 4 行 × 2 デッキ、さらにシフト 1st〜4th で 96 スロットあります。
この画面では

- デッキ TOP / BOTTOM を実機どおりの配置で並べる（液晶・フェーダー・キーパッド・ジョイスティック込み）
- 行は**バスへのデリゲート**（BGD-A / BGD-B / Layer-1 / Layer-2）として扱う
- 下の**割当一覧に 1st〜4th を横並びで表示**する
- XPT を選ぶと**その列の 4 段すべて**が右に出る
- 同じソースが複数スロットにあれば**重複として警告**する
- **パネル表示名を `#` 付きで編集**すると、キーの改行がその場で分かる

としています。仕込みを紙に書き出して現場に貼る運用も想定しています。

## 表記のルール

英字は大文字の略号（`PGM` `PST` `AUX` `XPT` `FTB`）、時間はフレーム（`30 F`）、
説明文はアプリ内に置かない。根拠は [docs/ui-research.md](docs/ui-research.md) に。

## 触ってみるところ

- PGM / PST のキー → タリーがマルチビューとパネル画面にも反映
- CONFIG > PANEL → SHIFT 1st〜4th を切り替える。下の一覧は 4 段を横並びで表示
- CONFIG > INPUTS → パネル表示名に `#` を入れるとキーの改行プレビューが変わる
- MACROS → ブロックを足すと右の LUA がその場で変わる
- マクロ画面：左のマクロを選ぶ → ブロックを追加 → 右の生成コマンドが即更新
- ステップのパラメーター（`PVW` `0.5s` など）を押すと値が変わる
- パネル画面：キーを選ぶ → 右で割り当て → 重複があれば警告

## 生成されるもの

MACROS 画面の OUTPUT は **REST** と **LUA** を切り替えられます。
REST は `KAIROS_RestAPI_17_J.pdf` に照らして書式を合わせてあります。

```
# KAIROS REST API v1.7   M07   OP → 2ボックス
# http://Kairos:password@192.168.10.20:1234
# Content-Type: application/merge-patch+json
# <uuid> は GET /scenes/Main の actions から解決

PATCH /scenes/Main/snapshots/SNAP1
{"state":"recall"}

PATCH /scenes/Main/Background
{"sourceB":"IP2"}

PATCH /scenes/Main/actions/<uuid>   # BgdMix:transition_auto
{"state":"play"}

# 4 WAIT : REST 未対応。マクロ側で実行
```

REST にエンドポイントがない操作（WAIT / GPO / TMC）はブロック一覧で `MACRO` と表示し、
出力ではコメントとして落とします。

## この先

- [ ] Core 200 との実接続（REST の書式は確定済み）
- [ ] LUA の関数名の確定（REST で賄えるため優先度は低い）
- [ ] 当スタジオの SDI ボード枚数・実入出力数の反映
- [ ] Deck A / Deck B の運用方針（デュアル Core を使うか）
- [ ] 設定の差分表示と書き込み

> 生成される LUA の**パス表記は資料で確定**していますが、**関数名はまだ仮**です。
> 詳細と未確認事項は [docs/kairos-facts.md](docs/kairos-facts.md) の 7 章に。
