# scripted-skill-generator

`scripted-skill-generator` は、定型処理を `scripts/` 配下の実行可能なスクリプトへ委譲する Agent Skill を作成・改善するためのメタスキルです。

AI に毎回同じ処理を自然言語で再構成させるのではなく、データ取得、変換、集計、検証などの再現性が重要な処理をスクリプト化し、AI は要件整理、判断、結果解釈、例外対応に集中することを目的としています。

## フォルダ構成

```text
.
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

- `SKILL.md`: スキル本体の説明と実行手順
- `scripts/scripted_skill_tool.py`: スクリプト活用型スキルの雛形生成・検証ツール
- `references/`: スクリプト化判断、実行環境、品質チェックの補足資料
- `assets/`: 生成されるスキルのテンプレート

## 機能的特徴

- 定型処理をスクリプト化すべきか、自然言語ワークフローに残すべきかを判断する
- Python または Node.js のスクリプトを含む Agent Skill の雛形を生成する
- `--dry-run` を基本に、安全に内容を確認してからファイルを作成できる
- 生成したスキルの構成、メタデータ、スクリプトの基本的な安全性を検証できる
- Windows と Linux(Ubuntu) の両方で扱いやすい設計を前提にしている

## ライセンス

このフォルダ内の内容は MIT License のもとで利用できます。詳細は [LICENSE](LICENSE) を参照してください。

## 免責事項

本スキルおよび同梱スクリプトは現状有姿で提供されます。生成されたスクリプトやスキルを利用する前に、実行内容、入出力、外部サービスへの接続、ファイル変更の有無を必ず確認してください。

利用によって発生した損害、データ消失、外部サービス上の操作結果について、作者およびコントリビューターは責任を負いません。
