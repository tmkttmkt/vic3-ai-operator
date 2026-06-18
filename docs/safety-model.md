# Safety Model

## 目的

Safety Modelは、AIが生成した操作をそのままゲームに適用しないための安全設計である。

Vic3では一つの操作が国家全体に影響する。自動操作は段階的に解放し、危険操作は必ず人間承認を要求する。

## 権限レベル

### Level 0: 提案のみ

AIは操作案を出すだけで、GUI実行はしない。

用途:

- 初期実験
- セーブ解析確認
- 方針比較

### Level 1: 低リスク操作の自動実行

許可例:

- 小規模建設追加
- 研究選択
- 一部の生産方式変更

制限:

- 1回の操作数を制限
- 財政赤字が一定以上なら停止
- 実行後に必ず検証

### Level 2: 中リスク操作の半自動実行

許可例:

- 消費税設定
- 建設キュー並び替え
- 複数州への建設

制限:

- 人間承認またはルールベース承認が必要
- state_diffで効果確認

### Level 3: 高リスク操作は承認必須

対象:

- 外交プレイ
- 宣戦布告
- 法律制定
- 全面動員
- 大規模な税率変更
- 建物削除
- 従属国化、併合

## 危険操作の定義

以下の性質を持つ操作は危険操作とする。

- 戻しにくい
- 長期的影響が大きい
- 戦争、革命、破産を引き起こす
- 多数のPopや州に影響する
- 現在のstate.jsonだけでは結果予測が難しい

## Validatorの役割

Validatorは、AIの出力を以下の観点で検査する。

1. JSONとして正しいか
2. Action Schemaに一致するか
3. 許可されたaction typeか
4. パラメータが上限を超えていないか
5. 対象の州・施設・技術が存在するか
6. 財政・政治・軍事リスクが上限内か
7. 人間承認が必要な操作を自動実行しようとしていないか

## Risk Policy例

```json
{
  "max_actions_per_turn": 5,
  "max_build_levels_per_action": 5,
  "max_total_build_levels_per_turn": 10,
  "block_if_debt_ratio_above": 0.75,
  "block_if_legitimacy_below": 25,
  "block_if_revolution_risk": true,
  "always_require_approval": [
    "declare_war",
    "start_diplomatic_play",
    "enact_law",
    "mobilize_army",
    "delete_building"
  ]
}
```

## Human Approval

承認が必要な場合、Executorは停止し、以下を表示する。

```json
{
  "requires_human_approval": true,
  "reason": "外交プレイ開始は高リスク操作である。",
  "proposed_action": {
    "type": "start_diplomatic_play",
    "target": "Austria",
    "wargoal": "leadership"
  }
}
```

## 監査ログ

全操作はログに残す。

```text
logs/audit/
  1841-05-12T120000-plan.json
  1841-05-12T120000-validator.json
  1841-05-12T120000-execution.log
  1841-05-12T120000-state-before.json
  1841-05-12T120000-state-after.json
```

## 失敗時の停止条件

以下の場合は自動操作を停止する。

- セーブ解析に失敗した
- Validatorが失敗した
- Vic3ウィンドウが見つからない
- 画面解像度が想定と違う
- 実行後検証に失敗した
- 同じ操作が2回連続失敗した
- 人間承認が必要な操作が含まれる

## 原則

AIを信用しない。AIの提案は候補であり、Validatorを通った操作だけが命令である。
