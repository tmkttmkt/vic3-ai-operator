# Roadmap

## Phase 0: Documentation First

目的: 設計思想、責務分離、安全設計を文書化する。

成果物:

- README
- Architecture
- Save Analysis
- Decision Layer
- GUI Execution
- Action Schema
- Safety Model

完了条件:

- システム全体のデータフローが説明できる
- 最小プロトタイプの範囲が明確である
- 危険操作の扱いが明確である

## Phase 1: Save Analysis Prototype

目的: `.v3`セーブを読み、AI向け`state.json`を作る。

実装対象:

- save_watcher.py
- save_extractor.py
- save_parser.py
- state_normalizer.py

最初に抽出する値:

- ゲーム日付
- プレイヤー国
- 財政状態
- 金準備
- 借金
- 建設キュー
- 不足商品
- 高騰商品

完了条件:

- 最新セーブを検出できる
- `.v3`を展開できる
- `state.example.json`相当のJSONを出力できる

## Phase 2: Decision Prototype

目的: `state.json`からAIが操作案を出す。

実装対象:

- minister prompt templates
- prime minister prompt template
- action_plan generation
- schema validation

最初の大臣AI:

- Industry Minister
- Economy Minister

完了条件:

- `state.json`から`action_plan.json`を生成できる
- JSON Schemaに通る
- 危険操作を出さない

## Phase 3: Validator

目的: AI出力を安全に制限する。

実装対象:

- action_schema.json
- risk_policy.json
- action_validator.py

完了条件:

- 不正JSONを弾ける
- 未許可操作を弾ける
- 危険操作に承認フラグを付けられる

## Phase 4: GUI Executor MVP

目的: 検証済みの建設操作を画面上で実行する。

最初の対応操作:

- build

想定シナリオ:

```text
鉄不足を検出
  -> iron_mine建設案生成
  -> Validator通過
  -> 建設画面を開く
  -> Iron Mineを検索
  -> 州を選択
  -> 建設キューへ追加
```

完了条件:

- 1種類の建物を指定州に追加できる
- 実行前後スクリーンショットを保存できる
- 失敗時に停止できる

## Phase 5: Verification Loop

目的: 実行結果を次回セーブ解析で確認する。

実装対象:

- state_diff.py
- verification report
- retry policy

完了条件:

- 実行前後の`state.json`を比較できる
- 建設キューへの追加を検出できる
- 成功/失敗をログ化できる

## Phase 6: Multi-Minister System

目的: 複数AIによる役割別提案と統合判断を実装する。

追加する大臣AI:

- Politics Minister
- Technology Minister
- Military Minister
- Diplomacy Minister

完了条件:

- 各大臣AIが個別提案を出せる
- Prime Minister AIが統合計画を出せる
- 矛盾した提案を棄却できる

## Phase 7: Semi-Autonomous Operation

目的: 低リスク操作を限定的に自動化する。

許可候補:

- 小規模建設
- 研究選択
- 一部生産方式変更

完了条件:

- Level 1操作を自動実行できる
- Level 2以上は承認待ちにできる
- 監査ログが残る

## Phase 8: Long-Run Evaluation

目的: 複数年のゲーム内運用を評価する。

評価指標:

- GDP成長
- 財政破綻の有無
- 建設効率
- 急進派増減
- 生活水準
- 戦争・革命の発生
- 操作失敗率

完了条件:

- ベースライン人間操作またはルールベース操作と比較できる
- 失敗パターンを分類できる

## 当面の最小目標

```text
セーブ解析 -> 鉄不足判定 -> 鉄鉱山建設案 -> Validator -> 手動承認 -> GUI実行 -> 次セーブで確認
```

この一本を通す。
