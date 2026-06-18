# vic3-ai-operator

Victoria 3を対象に、セーブデータ解析、AI判断、画面実行を分離して組み合わせるAI操作基盤の設計リポジトリである。

本プロジェクトは、LLMに直接マウス座標を考えさせるのではなく、以下の流れでゲーム操作を安定化する。

```text
save analysis -> state.json -> decision layer -> action_plan.json -> validator -> GUI executor -> verification
```

## 目的

最終目標は、人間が自然言語で国家方針を与え、複数のAIロールが大臣のように提案・判断し、承認済みの操作だけを画面実行する仕組みを作ることである。

想定する役割分担は以下である。

- 人間: 国家方針、最終承認、危険操作の判断
- 統合司令AI: 各大臣AIの提案統合、矛盾解消、優先順位付け
- 大臣AI: 経済、産業、政治、外交、軍事、技術などの担当別判断
- Validator: 操作許可、安全検査、危険操作の停止
- GUI Executor: pyautoguiや画像認識による画面操作
- Save Parser: Vic3セーブデータから状態を抽出
- Verifier: 実行結果をスクリーンショット/OCR/次回セーブで確認

## 設計原則

1. LLMに直接クリックさせない。
2. 判断と実行を分離する。
3. すべてのAI出力はJSON Schemaで検証する。
4. 危険操作は人間承認を必須にする。
5. 実行後は必ず状態差分で検証する。
6. 最初はIronmanを使わない。
7. 画面解像度、UIスケール、言語設定を固定する。

## 最小プロトタイプ

最初の到達目標は以下である。

```text
1. Vic3の.v3セーブを展開する
2. gamestateから日付・国・財政・建設キューを抽出する
3. state.jsonを作る
4. AIが次の建設案をaction_plan.jsonで出す
5. Validatorが危険操作を弾く
6. GUI Executorが建設画面を操作する
7. 次のセーブ解析で結果を確認する
```

## ドキュメント

- [Architecture](docs/architecture.md)
- [Save Analysis](docs/save-analysis.md)
- [Decision Layer](docs/decision-layer.md)
- [GUI Execution](docs/gui-execution.md)
- [Action Schema](docs/action-schema.md)
- [Safety Model](docs/safety-model.md)
- [Roadmap](docs/roadmap.md)

## 注意

このプロジェクトは実験用である。ゲーム操作自動化はバージョン差、UI変更、Mod、解像度、言語設定に強く影響される。最初から完全自律を目指さず、観測、提案、半自動操作、限定自動化の順に進める。
