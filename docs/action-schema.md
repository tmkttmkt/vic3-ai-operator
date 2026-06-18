# Action Schema

## 目的

Action Schemaは、AI判断結果をGUI Executorが安全に実行できる形式へ制限するための仕様である。

AIは自由文ではなく、必ず定義済みのJSON形式で操作を出す。

## action_plan.json

基本形は以下である。

```json
{
  "version": "0.1",
  "game_date": "1841.5.12",
  "goal": "鉄不足を緩和し、建設効率を改善する。",
  "summary": "財政赤字は許容範囲。鉄鉱山を優先する。",
  "actions": [
    {
      "id": "act-001",
      "type": "build",
      "priority": 1,
      "requires_human_approval": false,
      "reason": "鉄不足を緩和するため。",
      "params": {
        "building": "iron_mine",
        "state": "Silesia",
        "levels": 3
      }
    }
  ]
}
```

## 共通フィールド

### id

操作ID。ログ、検証、再試行で使う。

### type

操作種別。許可リストに含まれる必要がある。

### priority

実行順。小さいほど優先度が高い。

### requires_human_approval

人間承認が必要かどうか。

### reason

操作理由。Validatorや人間レビュー用。

### params

操作ごとの引数。

## 初期対応アクション

### build

建設キューへ施設を追加する。

```json
{
  "type": "build",
  "params": {
    "building": "iron_mine",
    "state": "Silesia",
    "levels": 3
  }
}
```

制約:

- `levels`は初期実装では1から5まで
- `building`は許可リストに存在すること
- `state`はセーブ解析上で存在すること

### change_research

研究対象を変更する。

```json
{
  "type": "change_research",
  "params": {
    "technology": "railways"
  }
}
```

制約:

- 研究候補に存在すること
- 進行中研究を中断する場合は理由が必要

### change_production_method

生産方式を変更する。

```json
{
  "type": "change_production_method",
  "params": {
    "building": "iron_mine",
    "state": "Silesia",
    "method_group": "mining_process",
    "method": "atmospheric_engine_pump"
  }
}
```

制約:

- 入力財の不足が悪化しすぎないこと
- 対象施設が存在すること
- 生産方式が解禁済みであること

### set_consumption_tax

消費税を設定する。

```json
{
  "type": "set_consumption_tax",
  "params": {
    "good": "services",
    "enabled": true
  }
}
```

制約:

- 権力コストを確認する
- 生活必需品への課税は人間承認を要求する

## 危険アクション

以下は初期実装では自動実行しない。

```json
[
  "declare_war",
  "start_diplomatic_play",
  "enact_law",
  "mobilize_army",
  "delete_building",
  "mass_change_production_method",
  "change_tax_level_high",
  "annex_subject"
]
```

これらが出力された場合、Validatorは`requires_human_approval`を強制的に`true`にする。

## Validator出力

```json
{
  "valid": true,
  "requires_human_approval": false,
  "accepted_actions": [],
  "blocked_actions": [],
  "warnings": []
}
```

## エラー例

```json
{
  "valid": false,
  "errors": [
    {
      "action_id": "act-002",
      "field": "params.levels",
      "message": "levels exceeds maximum allowed value 5"
    }
  ]
}
```

## 設計原則

Action Schemaは、AIの発想を機械が扱える命令へ圧縮する境界である。ここを曖昧にすると、実行層が破綻する。
