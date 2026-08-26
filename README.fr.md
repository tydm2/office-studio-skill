# office-studio — Workflow multi-agents pour documents de bureau

[English](README.md) · [简体中文](README.zh-CN.md) · [Español](README.es.md) · **[Français](README.fr.md)** · [Русский](README.ru.md) · [日本語](README.ja.md)



> Plateforme cible : **DSH** (cet environnement). Un pipeline unique pour créer/éditer efficacement trois types de documents de bureau — **Word · PPT · Excel** — avec des résultats axés sur la qualité et la prise en charge d'un **basculement en un clic entre plusieurs styles**.

## I. Pipeline

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

## II. Table de correspondance des mots déclencheurs

| Agent | Responsabilité en une ligne | Mots déclencheurs (langage naturel, y compris les variantes synonymes) |
|--------|-----------|------------------------------|
| **orchestrator** (Cerveau) | Prend un besoin → produit le style sélectionné + le lot de tâches autonome → agrège la livraison | Nouveau : `开始策划` `新建文档` `帮我做一份PPT` `帮我写一份Word` `帮我做一份Excel` `做文档` ; Édition : `编辑文档` `修改文档` `改这份PPT` `改这个Word` (privilégie la branche d'édition lorsque l'utilisateur fournit un fichier existant) |
| **word-crafter** | Prend le plan + le style → produit un .docx conforme aux spécifications | Nouveau : `撰写Word` `写正文` `生成Word` ; Édition : `改Word` `修改文档` `编辑这份Word` |
| **slide-designer** | Prend le plan + le style → produit un .pptx narratif | Nouveau : `制作PPT` `做幻灯片` `生成PPT` ; Édition : `改PPT` `修改这份PPT` |
| **sheet-analyst** | Prend les données + le plan + le style → produit un .xlsx axé d'abord sur la conclusion | Nouveau : `制作Excel` `做报表` `生成Excel` `分析数据` ; Édition : `改Excel` `修改表格` `编辑这份Excel` |

**Règles de déclenchement** : Les nouvelles tâches (sans archive de plan correspondante) passent toujours d'abord par `orchestrator` pour clarification ; une fois `plan-NN.md` confirmé par l'utilisateur, utilisez les mots déclencheurs dédiés de chaque expert pour y aller directement. Les mots déclencheurs n'entrent pas en conflit entre eux, ni avec les commandes système intégrées.

## III. Structure des répertoires

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

## IV. Contrats de fichiers (sortie amont → lecture aval)

1. **Planification → chaque expert (nouveau)** : `outputs/plans/plan-NN.md`, avec un format d'en-tête de métadonnées fixe :
   ```yaml
   ---
   doc_id: plan-NN
   title: <title>
   doc_type: word | ppt | excel
   style_id: <voir shared/style-catalog.md — 14 styles : 6 classiques + 8 inspirés de la communauté ; sélecteur dans shared/style-switcher.md>
   brand_kit: <optionnel ; écrire brand-NN lorsque l'utilisateur fournit des éléments de marque, règles dans shared/brand-kit.md ; omettre s'il n'y en a pas>
   audience: <audience>
   purpose: <purpose>
   length: <length>
   data_source: <optionnel, chemin du fichier de données Excel>
   ---
   ```
1b. **Tâche d'édition → chaque expert (édition)** : `outputs/plans/edit-NN.md` :
   ```yaml
   ---
   doc_id: edit-NN
   source_file: <chemin du document original, fourni/glissé par l'utilisateur>
   doc_type: word | ppt | excel
   mode: edit
   change_request: <demande de modification, spécifique au paragraphe/à la page/à la cellule>
   keep: <éléments à conserver impérativement : mise en forme d'origine / conclusions existantes / données>
   style_id: <par défaut, le style d'origine ; changer de style nécessite une spécification explicite de l'utilisateur>
   ---
   ```
2. **Sortie de chaque expert** : `outputs/<type>/<filename>`, accompagnée d'un `-meta.md` du même nom (enregistrant : doc_id, style_id, **brand_kit (le cas échéant)**, date de génération, source des données, notes de modification).
3. **Itération** : Tous les commentaires de révision de l'utilisateur passent par `send_message` de DSH pour poursuivre la conversation de l'agent atomique en vue de la révision, en préservant le contexte ; les révisions doivent être consignées dans les « notes de modification » du `-meta.md`, sans jamais réécrire silencieusement le contenu confirmé.
4. **Kit de marque (v2)** : Lorsque l'utilisateur fournit des éléments de marque (couleurs de marque / Logo / polices / modèles) → orchestrator les rassemble dans `shared/brand-kit.md`, le plan écrit `brand_kit: brand-NN`, et les trois experts les appliquent uniformément (la marque prime sur le catalogue de styles) ; lors de la livraison de plusieurs documents sous le même brand_id, orchestrator vérifie la cohérence de la marque (voir `shared/brand-kit.md`).

## V. Crédits communautaires (références de conception)

| Source | Points empruntés |
|------|--------|
| [anthropics/skills](https://github.com/anthropics/skills) (officiel) | Le paradigme docx/pptx/xlsx « lire le fichier existant → générer → vérifier qu'il s'ouvre » |
| [johnson7788/skill-ppt-agents](https://github.com/johnson7788/skill-ppt-agents) + [MultiAgentPPT](https://github.com/johnson7788/MultiAgentPPT) | PPT « plan → contenu → conception » en plusieurs étapes, avec validation indépendante à chaque étape |
| [rafalozan0/DocFlow](https://github.com/rafalozan0/DocFlow-Presentations-and-Docs-Skill) | python-pptx/docx/openpyxl unifiés pour le trio + répertoire de modèles basé sur des fichiers (basculement multi-styles) |
| [cabbage2000-lab/data-analysis-skills](https://github.com/cabbage2000-lab/data-analysis-skills) | Structure Excel en six paragraphes axée d'abord sur la conclusion, données traçables, aucune fabrication, corrélation ≠ causalité |
| [fleurytian/awesome-claude-skills](https://github.com/fleurytian/awesome-claude-skills) (ex-McKinsey) | Principe de la pyramide / structure narrative MECE |
| [hugohe3/ppt-master](https://github.com/hugohe3/ppt-master) (dépôt de spécifications de styles) | Le style Swiss Grid et d'autres spécifications de conception → stockés comme le style `swiss-grid` |
| [corazzon/pptx-design-styles](https://github.com/corazzon/pptx-design-styles) / [sunchaokun/PPT-Design-Skill](https://github.com/sunchaokun/PPT-Design-Skill) / [GordenSun/GordenPPTSkill](https://github.com/GordenSun/GordenPPTSkill) | Recherche sur les systèmes de styles PPT communautaires et la pratique des modèles (emprunt des seules spécifications de styles, sans copier les modèles) |
| [Tendances de conception 2025 de SlideRabbit](https://sliderabbit.com/blog/inspiring-2025-presentation-design-trends/) et autres | Tendances brutalisme / Y2K / dégradés → stockées comme `brutalist` / `retro-y2k` / `aurora-gradient` |
| [Storyset](https://storyset.com/) / [unDraw](https://undraw.co/) / [manypixels](https://www.manypixels.co/gallery) | Écosystème gratuit d'illustrations à plat → style `flat-illustration` et référence de correspondance pour la génération d'images |

Toutes les sources ci-dessus ont passé la revue de sécurité et de santé (injection de prompt / instructions malveillantes / exfiltration de données / licences de droits d'auteur / santé active, 2026-08-24) ; seule l'essence a été extraite, adaptée et réécrite dans la bibliothèque, sans copie textuelle ; toutes les entrées du catalogue de styles citent leurs liens sources.

## V·5 Réutilisation du blueprint

Blueprint existant : **办公文档** (`blueprints/办公文档.md`, contenant le schéma de topologie / la liste des agents / les enregistrements de décision ADR / les parties communautaires réutilisables / les leçons apprises). Pour des besoins similaires (workflows multi-agents de type document), vous pouvez dire 「**复用 办公文档 拓扑**」 — il suffit de clarifier les différences, sans reconcevoir ; les changements itératifs sont réécrits à la fois dans le blueprint et dans ce document.

## VI. Instructions de première exécution

1. Nouvelle tâche : dites simplement 「**帮我做一份[PPT/Word/Excel]**」 ou 「**开始策划**」, et orchestrator vous interrogera sur le sujet / le public / la longueur / le style, **et vous demandera s'il existe des éléments de marque (le cas échéant, produire uniformément selon le kit de marque)**.
2. Choisir le style : orchestrator demande d'abord une direction (professionnel sobre / moderne haut de gamme / personnalité marquée / contenu accessible), puis propose 3 à 4 options de style spécifiques (14 styles : Business professionnel / McKinsey / Académique / Minimaliste / Créatif / Officiel + Grille suisse / Glassmorphisme / Éditorial magazine / Tech sombre / Dégradé aurore / Illustration à plat / Brutaliste / Rétro Y2K), avec des liens vers les sources communautaires pour référence.
3. Après avoir confirmé le plan, dites le mot déclencheur de l'expert correspondant (par ex. 「制作PPT」) pour produire le document.
4. Modifier (un document déjà livré) : dites simplement quoi changer, et orchestrator utilisera `send_message` pour que l'agent atomique poursuive la conversation en vue de la révision.
5. **Pas satisfait du style ? Changez de tour** : dites simplement 「换一种风格/风格不满意/换个风格重做」 — le contenu et la structure sont entièrement préservés, orchestrator propose à nouveau des options de style, produit un nouveau plan avec un nouveau `style_id` et le refait, le nouveau fichier n'écrase pas l'ancien, et la livraison inclut une comparaison de styles avant/après. Recommandez au maximum 3 tours consécutifs, puis passez à l'affinage des détails ou fournissez une image de référence.
6. **Modifier un document existant** : envoyez-moi le chemin du .docx/.pptx/.xlsx (ou glissez-le) et dites 「**改这个Word / 改这份PPT / 修改表格** + quoi changer」, orchestrator identifie d'abord la structure/le style du document original, puis fait effectuer par l'expert des modifications incrémentales, et la livraison inclut une liste de modifications détaillée sans écraser le fichier original.

## VII. Barrière de sécurité

**Barrière de sécurité franchie (2026-08-23)** : Tous les fichiers AGENT.md et knowledge/ ont passé cinq revues — injection de prompt / instructions malveillantes / exfiltration de données / empoisonnement de la chaîne d'approvisionnement / sécurité de la plateforme ; le contenu des sources communautaires est traçable, avec les conclusions de revue jointes.
**Re-revue de la barrière de sécurité franchie (2026-08-24, après l'itération du kit de marque v2)** : Le fichier nouvellement ajouté `shared/brand-kit.md` et les sections de kit de marque de chaque AGENT.md ont passé la re-revue (aucune injection de prompt / instruction malveillante / exfiltration de données / résidu d'identifiants), et le registre des mots déclencheurs a été gelé sans modification.

## VIII. Améliorations des capacités PPT v3 (2026, distillées après l'étude de 7 plugins PPT communautaires)

> En empruntant les **mécanismes** (pas le code) de dsh-ppt / PPTKit Presentation / @yejiming/dsh-ppt / pptfast / pptwise / DeepSeek Design / dsh-univer-office, slide-designer ajoute quatre capacités :

- **Contrôle qualité par revue visuelle** : `agents/slide-designer/scripts/visual-qa.py` détecte automatiquement le débordement de texte / le chevauchement d'éléments / le contraste insuffisant / les éléments hors limites / les polices manquantes (`python visual-qa.py <deck.pptx>`) ; `knowledge/visual-qa.md` fournit une liste de vérification manuelle et des règles de correction automatique.
- **Collaboration avec l'écosystème de plugins** : `knowledge/plugin-ecosystem.md` consigne les commandes d'installation et les modes de collaboration de 7 plugins PPT communautaires (cette compétence produit la première ébauche narrative → les plugins gèrent le raffinement de la visualisation / la revue).
- **Correspondance des alias de thèmes** : `shared/style-catalog.md` ajoute des correspondances entre les noms de thèmes communautaires et les `style_id` (Data Drift→`dark-tech`, Swiss Pulse→`swiss-grid`, Velvet Standard→`editorial-magazine`, etc.).
- **Améliorations** : spécifications d'abord (les spécifications au niveau de la page sont finalisées avant le rendu), doubles artefacts d'aperçu HTML, extraction des couleurs/polices des PPT existants de l'entreprise vers le kit de marque.
