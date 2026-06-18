# Architecture

## 概要

vic3-ai-operatorは、Victoria 3の状態取得、AI判断、GUI実行を疎結合にした操作基盤である。

中核思想は「セーブデータで観測し、AIで判断し、画面操作で実行し、次の状態で検証する」ことである。

```text
+----------------+
| Human Operator |
+--------+-------+
         |
         v
+---------------------+
| Strategic Directive |
+----------+----------+
           |
           v
+---------------------+       +----------------+
| Prime Minister AI   |<----->| Minister AIs   |
+----------+----------+       +----------------+
           |
           v
+---------------------+
| action_plan.json    |
+----------+----------+
           |
           v
+---------------------+
| Action Validator    |
+----------+----------+
           |
           v
+---------------------+
| GUI Executor        |
+----------+----------+
           |
           v
+---------------------+
| Victoria 3          |
+----------+----------+
           |
           v
+---------------------+
| Save Parser         |
+----------+----------+
           |
           v
+---------------------+
| state.json          |
+---------------------+
```

## レイヤー

### 1. Save Analysis Layer

Vic3の`.v3`セーブファイルを読み、AIが扱いやすい`state.json`へ変換する。

責務:

- 最新セーブの検出
- `.v3`の展開
- `meta`と`gamestate`の読み取り
- 国家、経済、政治、軍事、外交の基本状態抽出
- 前回状態との差分生成

### 2. Decision Layer

`state.json`と人間の国家方針を入力し、複数AIが役割別に判断する。

責務:

- 状態要約
- 経済、産業、政治、軍事、外交、技術の担当別提案
- 提案の競合解消
- 操作計画`action_plan.json`の生成

### 3. Validation Layer

AIの出力をそのまま実行せず、許可された操作だけに制限する。

責務:

- JSON Schema検証
- 危険操作の検出
- 人間承認フラグ付与
- 不正な対象名、過剰な操作量、財政リスクの検出

### 4. GUI Execution Layer

検証済みの`action_plan.json`をVic3の画面操作に変換する。

責務:

- Vic3ウィンドウのアクティブ化
- 一時停止確認
- パネルを開く
- 検索、選択、クリック、入力
- スクリーンショット保存

### 5. Verification Layer

実行後に、操作が実際に反映されたかを確認する。

責務:

- 実行後スクリーンショット確認
- OCRによる簡易確認
- 次回セーブデータとの差分確認
- 成功、失敗、要再試行の判定

## 推奨ディレクトリ構成

```text
vic3_ai_operator/
  docs/
  schemas/
  src/
    save/
      watcher.py
      extractor.py
      parser.py
      normalizer.py
    decision/
      prime_minister.py
      ministers/
        economy.py
        industry.py
        politics.py
        diplomacy.py
        military.py
        technology.py
    validation/
      action_validator.py
      risk_policy.py
    execution/
      gui_executor.py
      commands.py
      screen.py
    verification/
      state_diff.py
      vision_check.py
    main_loop.py
  examples/
    state.example.json
    action_plan.example.json
  logs/
  tests/
```

## 実装方針

最初は完全自律ではなく、以下の順で段階化する。

1. セーブ解析のみ
2. AI提案のみ
3. 人間承認つき画面実行
4. 建設など低リスク操作の自動化
5. 状態差分による閉ループ化
6. 複数大臣AIによる統合判断

## 重要な制約

- Vic3のバージョン更新でセーブ構造やUIが変わる可能性がある。
- Mod環境ではキー名や状態構造が変化する可能性がある。
- GUI実行は解像度、UIスケール、言語設定に依存する。
- Ironmanは実験に向かない。
- AI出力は常に検証する必要がある。
