# Save Analysis

## 目的

Save Analysis Layerは、Victoria 3のセーブデータからゲーム状態を抽出し、AI判断用の`state.json`を生成する。

GUI画面からOCRだけで状態を読む方式は不安定である。可能な限りセーブデータを一次情報として扱い、画面認識は補助確認に限定する。

## 入力

- Vic3の`.v3`セーブファイル
- `meta`
- `gamestate`
- 必要に応じて前回の`state.json`

## 出力

- `state.json`
- `state_diff.json`
- 解析ログ

## 基本フロー

```text
save games folder
  -> latest .v3 detection
  -> unzip
  -> read meta/gamestate
  -> extract raw values
  -> normalize names and IDs
  -> state.json
  -> diff with previous state
```

## セーブ形式

Vic3の通常セーブは`.v3`拡張子を持つ。多くの場合zipとして展開でき、中に`meta`と`gamestate`が含まれる。

テキスト形式セーブを使うと解析しやすい。初期実験ではテキスト形式セーブを前提にする。

## 推奨設定

- Ironman: off
- Autosave: short interval
- Save format: text if possible
- Game language: English推奨
- Mods: 初期実験ではなし

## 抽出対象

### Meta

```json
{
  "game_date": "1840.1.1",
  "version": "...",
  "country": "JAP",
  "player_country": "JAP",
  "mods": []
}
```

### Country

```json
{
  "tag": "JAP",
  "rank": "minor_power",
  "prestige": 0,
  "gdp": 0,
  "population": 0,
  "literacy": 0
}
```

### Budget

```json
{
  "balance": 0,
  "income": 0,
  "expenses": 0,
  "gold_reserve": 0,
  "debt": 0,
  "credit_limit": 0,
  "tax_level": "normal"
}
```

### Economy

```json
{
  "construction_points": 0,
  "construction_queue": [],
  "shortages": [],
  "expensive_goods": [],
  "cheap_goods": [],
  "market_access_problems": []
}
```

### Politics

```json
{
  "legitimacy": 0,
  "radicals": 0,
  "loyalists": 0,
  "laws": [],
  "enacting_law": null,
  "interest_groups": []
}
```

### Military

```json
{
  "battalions": 0,
  "flotillas": 0,
  "mobilized": false,
  "wars": [],
  "fronts": []
}
```

## state.jsonの設計

AIが読むファイルは、ゲーム内部の完全構造ではなく、意思決定に必要な正規化済み情報にする。

```json
{
  "timestamp": "2026-06-18T00:00:00+09:00",
  "game": {
    "date": "1841.5.12",
    "version": "unknown"
  },
  "player": {
    "country": "JAP",
    "strategy": "industrialize_without_bankruptcy"
  },
  "budget": {
    "status": "deficit",
    "balance": -1200,
    "gold_reserve": 50000,
    "debt_ratio": 0.12
  },
  "economy": {
    "main_bottlenecks": ["iron", "tools"],
    "construction_queue_length": 12,
    "recommended_focus": ["iron_mine", "tooling_workshop"]
  },
  "risks": [
    "budget_deficit",
    "tools_shortage"
  ]
}
```

## 実装単位

### watcher.py

最新セーブファイルを検出する。

### extractor.py

`.v3`をzipとして展開し、`meta`と`gamestate`を取り出す。

### parser.py

`gamestate`から必要な値を抽出する。

### normalizer.py

内部IDや生データをAI向けの名前・単位・カテゴリに変換する。

### state_diff.py

前回状態と今回状態を比較し、操作が成功したかを検証する。

## パーサ方針

最初から完全なParadox構文パーサを作らない。まずは必要なキーだけを検索し、段階的に抽出範囲を増やす。

初期段階:

- 日付
- プレイヤー国
- 収支
- 金準備
- 建設キュー
- 商品不足

中期段階:

- 州ごとの施設
- 市場価格
- Pop雇用
- 生活水準
- 利益団体
- 法律

後期段階:

- 外交関係
- 戦争と戦線
- 技術
- 国家ランク
- 長期推移分析

## 検証

セーブ解析は、実行前後の差分確認に使う。

例:

```json
{
  "expected": {
    "construction_queue_added": ["iron_mine:Silesia:3"]
  },
  "actual": {
    "construction_queue_added": ["iron_mine:Silesia:3"]
  },
  "result": "success"
}
```

## 失敗時の扱い

- セーブ展開失敗: GUI実行を停止する
- 必須値抽出失敗: AI判断を停止する
- 差分確認失敗: 次の操作に進まない
- Mod由来の未知キー: warningとして保持する

## 原則

セーブ解析はAI操作システムの目である。目が腐ると判断も腐るため、不確かな値には必ず`confidence`を付ける。
