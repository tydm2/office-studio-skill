# office-studio — オフィス文書のためのマルチエージェントワークフロー

[English](README.md) · [简体中文](README.zh-CN.md) · [Español](README.es.md) · [Français](README.fr.md) · [Русский](README.ru.md) · **[日本語](README.ja.md)**



> 対象プラットフォーム: **DSH**（本環境）。3種類のオフィス文書 — **Word · PPT · Excel** — を、品質最優先で効率的に作成・編集するための単一パイプラインであり、**複数スタイルのワンクリック切り替え**にも対応しています。

## I. パイプライン

```
【新建】用户一句话（如"帮我做一份商务风PPT"）
        │
        ▼
orchestrator（大脑·策划调度）──澄清需求+选风格──▶ 产出 outputs/plans/plan-NN.md ──▶ 用户确认
        │                                              │
        │ 派发自包含任务包（含已选风格）                │
        ├──────────────┬──────────────┬────────────────┤
        ▼              ▼              ▼                ▼
   word-crafter   slide-designer  sheet-analyst   （按 doc_type 只派一个）
   （Word撰写·小组） （PPT设计·小组） （Excel报表·单一资深）
        ▼              ▼              ▼
   outputs/word/   outputs/ppt/    outputs/excel/
   doc-NN.docx     deck-NN.pptx    sheet-NN.xlsx
        └──────────────┴──────────────┴────────────────┘
                        │ 汇总汇报
                        ▼
                    orchestrator → 交付用户 → 迭代（send_message 续聊修订）

【编辑】拖入/提供现有文档（.docx/.pptx/.xlsx 路径）
        │
        ▼
orchestrator ──读取识别（结构/风格/内容概要）──▶ 澄清 修改目标/保留项/风格
        │
        ▼ 产出 outputs/plans/edit-NN.md（source_file + change_request + keep）
   对应专家：先读原文件 → 增量修改（保留原结构风格）→ 产出 doc/deck/sheet-NN + 逐条改动清单
        │
        ▼
   交付用户 → 迭代（send_message 续聊修订）
```

## II. トリガーワード対応表

| エージェント | 一行の責務 | トリガーワード（自然言語、同義のバリエーションを含む） |
|--------|-----------|------------------------------|
| **orchestrator**（頭脳） | 要件を受け取り → 選択したスタイル + 自己完結型タスクパッケージを出力 → 成果物を集約して納品 | 新規: `开始策划` `新建文档` `帮我做一份PPT` `帮我写一份Word` `帮我做一份Excel` `做文档`; 編集: `编辑文档` `修改文档` `改这份PPT` `改这个Word`（ユーザーが既存ファイルを提供した場合は編集ブランチを優先） |
| **word-crafter** | プラン + スタイルを受け取り → 仕様準拠の .docx を出力 | 新規: `撰写Word` `写正文` `生成Word`; 編集: `改Word` `修改文档` `编辑这份Word` |
| **slide-designer** | プラン + スタイルを受け取り → 物語性のある .pptx を出力 | 新規: `制作PPT` `做幻灯片` `生成PPT`; 編集: `改PPT` `修改这份PPT` |
| **sheet-analyst** | データ + プラン + スタイルを受け取り → 結論ファーストの .xlsx を出力 | 新規: `制作Excel` `做报表` `生成Excel` `分析数据`; 編集: `改Excel` `修改表格` `编辑这份Excel` |

**トリガールール**: 新規タスク（対応するプランの記録がないもの）は、必ず最初に `orchestrator` に入って要件を明確化します。ユーザーが `plan-NN.md` を確定した後は、各エキスパート専用のトリガーワードを使って直接進めます。トリガーワードは相互に競合せず、組み込みのシステムコマンドとも競合しません。

## III. ディレクトリ構成

```
office-studio/
  README.md                      # 本文件：导航 + 流水线图 + 触发词映射
  shared/
    style-catalog.md             # 刷新型风格库（14 种风格：6 经典 + 8 社区借鉴，多风格切换核心）
    style-switcher.md            # 风格选择器 + 不满意换一轮协议
    brand-kit.md                 # ★品牌套件（v2）：跨文档品牌统一——同一客户的 Word/PPT/Excel 共用品牌色/字体/Logo/页眉页脚
  blueprints/
    办公文档.md                    # 领域拓扑沉淀（复用与决策记录，v1.6 机制）
  feedback-log.md                # 工作流级需求记忆（运行期迭代用）
  usage-log.md                   # 工作流级使用留痕（运行期迭代用）
  agents/
    orchestrator/                # 大脑（策划/调度）
      AGENT.md
      knowledge/                 # task-intake / quality-redlines / community-refs
      references/                # feedback-log / usage-log
    word-crafter/                # Word 撰写专家（专家小组制）
      AGENT.md
      knowledge/                 # craft-methodology / style-application / quality-checklist / output-template / community-refs
      references/
    slide-designer/              # PPT 设计专家（专家小组制）
      AGENT.md
      knowledge/                 # narrative-methodology / style-application / quality-checklist / output-template / community-refs / visual-qa / plugin-ecosystem
      scripts/visual-qa.py       # 视觉审稿质检脚本（溢出/重叠/对比度/越界/字体）
      references/
    sheet-analyst/               # Excel 报表专家（单一资深专家）
      AGENT.md
      knowledge/                 # analysis-methodology / style-application / quality-checklist / output-template / community-refs
      references/
  outputs/
    plans/                       # 策划任务包 plan-NN.md / 编辑任务包 edit-NN.md
    word/                        # doc-NN.docx + doc-NN-meta.md
    ppt/                         # deck-NN.pptx + deck-NN-meta.md
    excel/                       # sheet-NN.xlsx + sheet-NN-meta.md
```

## IV. ファイル契約（上流の出力 → 下流の読み取り）

1. **企画 → 各エキスパート（新規）**: `outputs/plans/plan-NN.md`。固定のヘッダーメタデータ形式は以下のとおりです:
   ```yaml
   ---
   doc_id: plan-NN
   title: <title>
   doc_type: word | ppt | excel
   style_id: <shared/style-catalog.md 参照 — 14スタイル: クラシック6 + コミュニティ由来8; セレクターは shared/style-switcher.md>
   brand_kit: <任意; ユーザーがブランド素材を提供した場合は brand-NN を記入（規則は shared/brand-kit.md）; なければ省略>
   audience: <audience>
   purpose: <purpose>
   length: <length>
   data_source: <任意、Excelデータファイルのパス>
   ---
   ```
1b. **編集タスク → 各エキスパート（編集）**: `outputs/plans/edit-NN.md`:
   ```yaml
   ---
   doc_id: edit-NN
   source_file: <ユーザーが提供・ドラッグインした元文書のパス>
   doc_type: word | ppt | excel
   mode: edit
   change_request: <変更要求。段落/ページ/セル単位で具体的に>
   keep: <保持すべき項目: 元の書式 / 既存の結論 / データ>
   style_id: <元のスタイルを既定値とする; スタイルの切り替えはユーザーの明示的な指定が必要>
   ---
   ```
2. **各エキスパートの出力**: `outputs/<type>/<filename>`。同名の `-meta.md` を添付します（記録内容: doc_id、style_id、**brand_kit（あれば）**、生成日、データソース、変更メモ）。
3. **反復（イテレーション）**: ユーザーのすべての修正コメントは、DSH の `send_message` を通じて当該エージェントの会話を継続し、コンテキストを保ったまま修正します。修正内容は必ず `-meta.md` の「変更メモ」に記録し、確定済みの内容を黙って書き換えてはいけません。
4. **ブランドキット（v2）**: ユーザーがブランド素材（ブランドカラー / ロゴ / フォント / テンプレート）を提供した場合 → orchestrator がそれらを `shared/brand-kit.md` に集約し、プランには `brand_kit: brand-NN` を記入し、3人のエキスパートが統一的に適用します（ブランドはスタイルカタログより優先）。同じ brand_id で複数の文書を納品する際、orchestrator はブランドの一貫性を検証します（`shared/brand-kit.md` 参照）。

## V. コミュニティへの謝辞（デザイン参考）

| 出典 | 借用したポイント |
|------|--------|
| [anthropics/skills](https://github.com/anthropics/skills)（公式） | docx/pptx/xlsx の「既存ファイルを読む → 生成 → 開けるか検証」というパラダイム |
| [johnson7788/skill-ppt-agents](https://github.com/johnson7788/skill-ppt-agents) + [MultiAgentPPT](https://github.com/johnson7788/MultiAgentPPT) | PPT の「アウトライン → 内容 → デザイン」という多段階構成（各段階で独立した受け入れ判定付き） |
| [rafalozan0/DocFlow](https://github.com/rafalozan0/DocFlow-Presentations-and-Docs-Skill) | 3種の文書に共通の python-pptx/docx/openpyxl + ファイルベースのテンプレートディレクトリ（マルチスタイル切り替え） |
| [cabbage2000-lab/data-analysis-skills](https://github.com/cabbage2000-lab/data-analysis-skills) | Excel の結論ファーストの6段落構成、トレーサブルなデータ、捏造なし、相関≠因果 |
| [fleurytian/awesome-claude-skills](https://github.com/fleurytian/awesome-claude-skills)（元マッキンゼー） | ピラミッド原則 / MECE のナラティブ構造 |
| [hugohe3/ppt-master](https://github.com/hugohe3/ppt-master)（スタイル仕様リポジトリ） | スイスグリッドスタイルほかのデザイン仕様 → `swiss-grid` スタイルとして格納 |
| [corazzon/pptx-design-styles](https://github.com/corazzon/pptx-design-styles) / [sunchaokun/PPT-Design-Skill](https://github.com/sunchaokun/PPT-Design-Skill) / [GordenSun/GordenPPTSkill](https://github.com/GordenSun/GordenPPTSkill) | コミュニティのPPTスタイル体系とテンプレート実践の調査（テンプレートの複製ではなく、スタイル仕様のみ借用） |
| [SlideRabbit 2025年デザイントレンド](https://sliderabbit.com/blog/inspiring-2025-presentation-design-trends/) ほか | ブルータリズム / Y2K / グラデーションのトレンド → `brutalist` / `retro-y2k` / `aurora-gradient` として格納 |
| [Storyset](https://storyset.com/) / [unDraw](https://undraw.co/) / [manypixels](https://www.manypixels.co/gallery) | 無料のフラットイラストレーションエコシステム → `flat-illustration` スタイルおよび画像生成マッピングの参考 |

上記のすべての出典は、セキュリティおよび健全性レビュー（プロンプトインジェクション / 悪意ある指示 / データ流出 / 著作権ライセンス / 稼働状況、2026-08-24）を通過しています。本質のみを抽出・改変・再構成してライブラリに収録しており、一字一句の複製は行っていません。スタイルカタログの各項目には出典リンクを明記しています。

## V·5 ブループリントの再利用

既存ブループリント: **办公文档**（`blueprints/办公文档.md`。トポロジ図 / エージェント一覧 / ADR 決定記録 / コミュニティ再利用パーツ / 得られた教訓を含む）。類似のニーズ（文書型マルチエージェントワークフロー）では、「**复用 办公文档 拓扑**」と言うだけで、差分を明確化するだけで再設計は不要です。反復的な変更は、ブループリントと本文書の両方に書き戻されます。

## VI. 初回実行手順

1. 新規タスク: 「**帮我做一份[PPT/Word/Excel]**」または「**开始策划**」と言うだけで、orchestrator がテーマ / 対象読者 / 分量 / スタイルについて質問し、**さらにブランド素材の有無を確認します（あればブランドキットに従って統一的に制作）**。
2. スタイルの選択: orchestrator はまず方向性（堅実プロフェッショナル / モダン高級 / 個性派 / 親しみやすいコンテンツ）を尋ね、その後、具体的なスタイル候補を3〜4つ提示します（14スタイル: ビジネスプロフェッショナル / マッキンゼー / アカデミック / ミニマル / クリエイティブ / オフィシャル + スイスグリッド / グラスモーフィズム / マガジンエディトリアル / ダークテック / オーロラグラデーション / フラットイラストレーション / ブルータリスト / レトロY2K）。参考としてコミュニティの出典リンクも提示します。
3. プランを確定した後、対応するエキスパートのトリガーワード（例: 「制作PPT」）を言うと文書が生成されます。
4. 修正（納品済み文書）: 変更したい内容を言うだけで、orchestrator が `send_message` を使って当該エージェントに会話を継続させ、修正します。
5. **スタイルが気に入らない？ 切り替えラウンド**: 「换一种风格/风格不满意/换个风格重做」と言うだけです。内容と構成は完全に保持され、orchestrator がスタイル候補を再提示し、新しい `style_id` で新しいプランを作成して作り直します。新しいファイルは古いファイルを上書きせず、納品時にはスタイルの変更前後の比較を含めます。連続ラウンドは最大3回を推奨し、その後は細部の微調整に切り替えるか、参考画像を提供してください。
6. **既存文書の編集**: .docx/.pptx/.xlsx のパスを送る（またはドラッグインする）か、「**改这个Word / 改这份PPT / 修改表格** + 変更内容」と言ってください。orchestrator がまず元文書の構造・スタイルを特定し、エキスパートが増分的に変更を加え、納品時には元ファイルを上書きせず、逐条の変更リストを含めます。

## VII. セキュリティゲート

**セキュリティゲート通過（2026-08-23）**: すべての AGENT.md ファイルおよび knowledge/ は、プロンプトインジェクション / 悪意ある指示 / データ流出 / サプライチェーン汚染 / プラットフォームセキュリティの5つのレビューを通過しました。コミュニティ由来のコンテンツは追跡可能で、レビュー結論が添付されています。
**セキュリティゲート再レビュー通過（2026-08-24、v2 ブランドキットの反復後）**: 新たに追加された `shared/brand-kit.md` と各 AGENT.md のブランドキット関連セクションは再レビューを通過しました（プロンプトインジェクション / 悪意ある指示 / データ流出 / 認証情報の残存なし）。トリガーワード登録簿は変更なく凍結されています。

## VIII. v3 PPT 機能強化（2026年、コミュニティのPPTプラグイン7種を調査して抽出）

> dsh-ppt / PPTKit Presentation / @yejiming/dsh-ppt / pptfast / pptwise / DeepSeek Design / dsh-univer-office の**メカニズム**（コードではなく）を借用し、slide-designer は4つの機能を追加します:

- **ビジュアルレビューQA**: `agents/slide-designer/scripts/visual-qa.py` が、テキストのあふれ / 要素の重なり / コントラスト不足 / 範囲外 / フォント欠落を自動検出します（`python visual-qa.py <deck.pptx>`）。`knowledge/visual-qa.md` には手動検査のチェックリストと自動修正ルールが用意されています。
- **プラグインエコシステム連携**: `knowledge/plugin-ecosystem.md` に、コミュニティのPPTプラグイン7種のインストールコマンドと連携方式を記録しています（本スキルが物語性のある初稿を生成 → プラグインがビジュアルの洗練 / レビューを担当）。
- **テーマ別名マッピング**: `shared/style-catalog.md` に、コミュニティのテーマ名から `style_id` へのマッピングを追加しています（データドリフト→`dark-tech`、スイスパルス→`swiss-grid`、ベルベットスタンダード→`editorial-magazine` など）。
- **機能強化**: スペックファースト（レンダリング前にページ単位の仕様を確定）、HTMLプレビューの2種生成、企業の既存PPTから色・フォントを抽出してブランドキットへ反映。
