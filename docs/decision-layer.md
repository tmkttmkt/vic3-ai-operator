# Decision Layer

## 目的

Decision Layerは、`state.json`と人間の方針を入力として、次に実行すべき操作案を`action_plan.json`として出力する。

LLMの仕事は判断であり、直接GUIを操作することではない。画面操作はExecutorへ渡す。

## 基本構造

```text
state.json
  + human_directive.md
  -> minister AIs
  -> proposals
  -> prime minister AI
  -> action_plan.json
  -> validator
```

## 役割AI

### Prime Minister AI

全体統合役である。

責務:

- 人間の国家方針を保持する
- 各大臣AIの提案を比較する
- 財政、政治、軍事、外交の矛盾を解消する
- 優先順位を決める
- 危険操作を承認待ちにする

### Economy Minister AI

財政と市場を担当する。

判断対象:

- 収支
- 借金
- 金準備
- 税率
- 消費税
- 高騰商品
- 不足商品

### Industry Minister AI

建設と生産を担当する。

判断対象:

- 建設キュー
- 建設力
- 州ごとの施設
- 生産方式
- 原材料不足
- インフラ

### Politics Minister AI

国内政治を担当する。

判断対象:

- 正統性
- 急進派
- 体制派
- 利益団体
- 法律
- 革命リスク

### Diplomacy Minister AI

外交を担当する。

判断対象:

- 関係
- 悪名
- 同盟
- 従属国
- 外交プレイ

### Military Minister AI

軍事を担当する。

判断対象:

- 大隊
- 艦隊
- 戦争
- 戦線
- 動員
- 装備不足

### Technology Minister AI

技術と研究を担当する。

判断対象:

- 現在研究
- 研究候補
- 国家方針との整合性
- 生産方式解禁

## 入力

```json
{
  "state": {},
  "human_directive": "10年以内に工業化する。ただし破産は禁止。",
  "role_policy": {
    "allow_autonomous_building": true,
    "allow_autonomous_diplomacy": false,
    "allow_autonomous_war": false
  }
}
```

## 大臣AIの出力

各大臣AIは提案のみを出す。

```json
{
  "role": "industry_minister",
  "assessment": "鉄と工具が高騰しており建設効率が悪化している。",
  "proposals": [
    {
      "priority": 1,
      "action": {
        "type": "build",
        "building": "iron_mine",
        "state": "Silesia",
        "levels": 3
      },
      "reason": "鉄不足を緩和するため。",
      "risk": "短期的に建設費が増える。",
      "confidence": 0.72
    }
  ]
}
```

## Prime Minister AIの出力

統合後の操作計画を出す。

```json
{
  "goal": "鉄不足を緩和し、建設効率を改善する。",
  "summary": "財政赤字は許容範囲。低リスク建設を優先する。",
  "actions": [
    {
      "id": "act-001",
      "type": "build",
      "building": "iron_mine",
      "state": "Silesia",
      "levels": 3,
      "priority": 1,
      "reason": "鉄不足の緩和。",
      "requires_human_approval": false
    }
  ],
  "rejected_proposals": [
    {
      "source": "military_minister",
      "reason": "現在は戦争計画がなく、軍拡より財政安定が優先。"
    }
  ]
}
```

## 判断制約

AIは以下を守る。

- 不明な値を確定情報として扱わない。
- セーブ解析で得られない情報は`unknown`とする。
- 危険操作は必ず`requires_human_approval: true`にする。
- 操作量は上限を持つ。
- 一度に多数の変更を行わない。

## 危険操作

以下は原則として人間承認を必要とする。

- 宣戦布告
- 外交プレイ開始
- 法律制定開始
- 全面動員
- 徴兵拡大
- 大規模借金を伴う建設
- 税率の大幅変更
- 建設キューの大規模削除
- 施設削除
- 従属国化、併合、条約港取得

## 初期プロンプト方針

大臣AIには担当範囲を明確に制限する。

例:

```text
あなたはVictoria 3の産業大臣AIである。
入力されたstate.jsonだけを根拠に、建設と生産方式に関する提案を最大3つ出せ。
外交、戦争、法律変更、税率変更は提案してはならない。
出力はJSONのみとする。
```

## 評価指標

Decision Layerの品質は以下で評価する。

- JSONがSchemaに通るか
- 危険操作を勝手に出していないか
- state.jsonに存在しない値を根拠にしていないか
- 操作後に目的へ近づいたか
- 同じ失敗を繰り返していないか

## ログ

すべての判断は保存する。

```text
logs/decisions/1841-05-12T120000/
  input_state.json
  human_directive.md
  economy_proposal.json
  industry_proposal.json
  prime_minister_plan.json
  validator_result.json
```

## 原則

AI判断は行政文書である。曖昧な自由文ではなく、検証可能なJSONとして残す。
