# scripted-skill-generator

`scripted-skill-generator` は、Agent Skill の中で **「自然言語で確率論的に（柔軟に）扱うべき処理」** と **「`scripts/` のスクリプトで確率論を排除して決定論的に（確実に）固定すべき処理」** の境界を AI が自律的に設計し、スキルを作成・改善するためのメタスキルです。

AI に毎回同じ定型処理を自然言語から再構成させることによるブレ・計算ミス・トークン消費を防ぎつつ、変わりやすい条件までスクリプトに直書きして硬直化してしまう「固定しすぎ」も防ぐことを目的としています。

## 設計思想：3層分離モデルと AI 自律判断

「全部スクリプト」か「全部自然言語」かの二択ではなく、処理を **3 つの層** に分けて設計します。

| 層 | 性質 | 役割 | 実装場所 |
|----|------|------|----------|
| **層1: 決定論コア（固定）** | 確率論を排除（再現性100%） | 取得、パース、計算・集計、ソート、規約チェック、固定形式出力、上書き保護 | `scripts/` |
| **層2: 可変境界（調整つまみ・契約）** | 固定と柔軟の橋渡し | 期間、対象パス、除外条件、出力形式、AI が生成する中間データ（JSON/Markdown） | CLI引数・設定・中間ファイル |
| **層3: 判断・解釈・対話（柔軟）** | 確率論的推論（文脈依存） | 曖昧な要望整理、構成設計、オプション選定、異常値・警告の解釈、改善提案 | `SKILL.md`（自然言語） |

また、境界の切り分けや設計判断をいちいちユーザーに委ねず、**AI が自律的に工程分解・境界判定・作成（改善）まで完走**します。ユーザーへ確認するのは「要件が根本的に不明」「新規インストール」「外部送信・認証・破壊的操作」など**本当に確認が必要なケースのみ**に絞り、完了時に「境界判断サマリー」を提示して必要に応じて事後調整できる形をとります。

## 対応する 4 つのアーキテクチャパターンと 2 つのモード

業務特性に応じて、次の 4 パターンから最適な構成を自律選定します。

- **パターンA（決定論パイプライン型）**: 取得 → 正規化 → 集計 → 固定形式出力までをスクリプトで一貫実行
- **パターンB（AI設計 ＋ スクリプト実行型）**: AI が柔軟に構成した中間データ（JSON/Markdown）をスクリプトが決定論的に検証・変換・描画
- **パターンC（スクリプト収集・検証 ＋ AI判断型）**: スクリプトが網羅的に事実収集・規約検出を行い、AI が優先度判定や改善提案を実施
- **パターンD（自然言語先行型）**: 仕様未確定・単発・判断中心の業務では無理に `scripts/` を作らず、同じ定型処理が2回以上発生したときの昇格条件のみを記載

また、以下の 2 モードに対応しています。

- **Mode 1: 新規作成（Create）** — 依頼内容と環境から自律的に設計・雛形生成・実装・検証まで実行
- **Mode 2: 既存スキルの改善・境界見直し（Refactor / Rebalance）** — 既存スキルの「固定しすぎ（ハードコード・脆い正規表現等）」と「柔軟すぎ（AI による暗算・目視集計等）」を診断してリバランス

## フォルダ構成

```text
.
├── .gitignore
├── SKILL.md
├── LICENSE
├── README.md
├── assets/
│   └── scripted-skill-template.md
├── references/
│   ├── runtime-policy.md
│   ├── script-first-design.md
│   └── script-quality-checklist.md
└── scripts/
    └── scripted_skill_tool.py
```

- `SKILL.md`: スキル本体の設計方針、自律判断ルール、新規作成・改善手順
- `scripts/scripted_skill_tool.py`: スクリプト活用型スキルの雛形生成（`scaffold`）・静的検証（`validate`）ツール
- `references/script-first-design.md`: 3層分離モデル、4軸判定、固定しすぎ／柔軟すぎの防止ガイド
- `references/runtime-policy.md`: Windows / Linux（Ubuntu）のランタイム確認、文字コード対策、インストール確認方針
- `references/script-quality-checklist.md`: 境界設計・実装品質・安全性・テストのチェックリスト
- `assets/scripted-skill-template.md`: 生成されるスクリプト活用型スキルのテンプレート

## 機能的特徴

- 3層分離モデルと逆方向セルフレビュー（固定しすぎ警報／柔軟すぎ警報）により、正確さと柔軟性を両立する境界を自律設計
- Python 標準ライブラリまたは Node.js（`.mjs`）によるクロスプラットフォーム対応（Windows / Linux(Ubuntu)）の雛形を `--pattern`（`pipeline` / `ai-to-script` / `script-to-ai`）別に生成
- 生成テンプレートは Python / Node.js ともに CSV・JSON 出力、`--dry-run` による無書き込み確認、`--force` なしでの既存出力ファイル保護、入力存在チェック、UTF-8 配慮、stdout/stderr 分離を標準装備
- `validate` は Python インプロセスの構文チェック（`compile()`）により `__pycache__` を生成しない副作用ゼロの検証を実現し、リンク切れ・BOM・ディレクトリ名不一致・未置換テンプレート／汎用トリガー残留・危険パターン・環境固有パスを自動検出
- パターンD（自然言語先行型）向けの `--allow-no-scripts` 検証オプションに対応

## 同梱ツールの主な使い方

### 1. スキル雛形の生成（ドライラン → 適用）

```bash
# まずドライランで作成予定を確認
python scripts/scripted_skill_tool.py scaffold \
  --name github-activity-exporter \
  --output .agents/skills \
  --purpose "GitHubの活動履歴を集計しCSV/JSONで出力する" \
  --script-name collect_activity \
  --runtime python \
  --pattern pipeline \
  --triggers "GitHub活動集計,GitHubアクティビティ出力"

# 問題なければ --apply を付けて作成
python scripts/scripted_skill_tool.py scaffold \
  --name github-activity-exporter \
  --output .agents/skills \
  --purpose "GitHubの活動履歴を集計しCSV/JSONで出力する" \
  --script-name collect_activity \
  --runtime python \
  --pattern pipeline \
  --triggers "GitHub活動集計,GitHubアクティビティ出力" \
  --apply
```

### 2. スキルの静的検証

```bash
# スクリプト活用型スキルの検証（__pycache__ を生成しません）
python scripts/scripted_skill_tool.py validate --skill .agents/skills/github-activity-exporter

# 自然言語先行型（パターンD）スキルの検証
python scripts/scripted_skill_tool.py validate --skill .agents/skills/my-nl-skill --allow-no-scripts
```

## ライセンス

このフォルダ内の内容は MIT License のもとで利用できます。詳細は [LICENSE](LICENSE) を参照してください。

## 免責事項

本スキルおよび同梱スクリプトは現状有姿で提供されます。生成されたスクリプトやスキルを利用する前に、実行内容、入出力、外部サービスへの接続、ファイル変更の有無を必ず確認してください。

利用によって発生した損害、データ消失、外部サービス上の操作結果について、作者およびコントリビューターは責任を負いません。
