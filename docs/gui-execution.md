# GUI Execution

## 目的

GUI Execution Layerは、検証済みの`action_plan.json`をVictoria 3の画面操作へ変換する。

LLMが直接マウス座標を決める設計は禁止する。LLMは意図をJSONで出し、Executorが具体的な操作手順を持つ。

## 基本フロー

```text
action_plan.json
  -> validator
  -> command mapping
  -> window activation
  -> pause game
  -> open target panel
  -> execute operation
  -> screenshot
  -> verification
```

## 実行環境の固定

初期実験では以下を固定する。

- 画面解像度: 1920x1080
- UIスケール: 固定
- ゲーム言語: English推奨
- ウィンドウモード: BorderlessまたはWindowed
- ゲーム速度: 実行中は一時停止
- Ironman: off

UI自動化は環境差に弱い。ここを固定しないまま高度化すると、座標がズレた瞬間に国家が崩れる。

## Executorの責務

- Vic3ウィンドウをアクティブ化する
- ゲームを一時停止する
- パネルを開く
- 検索欄へ入力する
- 対象州や施設を選択する
- ボタンを押す
- 実行後スクリーンショットを保存する
- 操作ログを残す

## Command API

Executorは以下のような高レベルコマンドを受け取る。

```json
{
  "type": "build",
  "building": "iron_mine",
  "state": "Silesia",
  "levels": 3
}
```

内部では以下の手順へ変換する。

```text
open_building_panel()
search_building("iron_mine")
select_state("Silesia")
add_to_construction_queue(3)
confirm_queue_changed()
```

## 初期対応コマンド

### build

建設キューへ施設を追加する。

```json
{
  "type": "build",
  "building": "tooling_workshop",
  "state": "Kanto",
  "levels": 2
}
```

### change_research

研究対象を変更する。

```json
{
  "type": "change_research",
  "technology": "railways"
}
```

### change_production_method

施設の生産方式を変更する。

```json
{
  "type": "change_production_method",
  "building": "iron_mine",
  "state": "Silesia",
  "method_group": "mining_process",
  "method": "atmospheric_engine_pump"
}
```

## 実装候補

### pyautogui

最初の実装に向く。

用途:

- キーボードショートカット
- クリック
- テキスト入力
- スクリーンショット

### OpenCV

UI要素検出に使う。

用途:

- ボタンテンプレート検出
- アイコン検出
- 画面状態判定

### OCR

補助確認に使う。

用途:

- 検索結果名の確認
- パネル名の確認
- エラー表示の検出

OCRを一次情報にしない。一次情報はセーブ解析である。

## 座標管理

座標はコードへ直書きせず、設定ファイルに分離する。

```json
{
  "resolution": "1920x1080",
  "ui_scale": "100%",
  "buttons": {
    "construction_panel": [120, 220],
    "search_box": [400, 180],
    "add_level_button": [1580, 820]
  }
}
```

## 実行前チェック

Executorは操作前に以下を確認する。

- Vic3ウィンドウが存在する
- 想定解像度である
- ゲームが一時停止している
- action_planがValidatorを通過している
- 直前操作が未検証のまま残っていない

## 実行後チェック

実行後は以下を保存する。

```text
logs/execution/<timestamp>/
  action_plan.json
  executor.log
  before.png
  after.png
  vision_result.json
```

さらに次回セーブ解析で、操作が反映されたか確認する。

## リトライ方針

失敗時に無限リトライしない。

- 1回目: 同じ操作を再試行
- 2回目: スクリーンショット保存して停止
- 3回目以降: 人間承認待ち

## 禁止事項

- LLMに座標を直接出させる
- 未検証のaction_planを実行する
- ゲーム進行中に複雑操作を行う
- 危険操作を自動実行する
- 画面確認なしで連続クリックする

## 原則

Executorは手であり、脳ではない。判断しない。命令を検証済みコマンドとして受け取り、機械的に実行する。
