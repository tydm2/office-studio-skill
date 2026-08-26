# office-studio — Flujo de trabajo multiagente para documentos de oficina

[English](README.md) · [简体中文](README.zh-CN.md) · **[Español](README.es.md)** · [Français](README.fr.md) · [Русский](README.ru.md) · [日本語](README.ja.md)



> Plataforma de destino: **DSH** (este entorno). Una única canalización para crear/editar de forma eficiente tres tipos de documentos de oficina — **Word · PPT · Excel** — con resultados orientados a la calidad y soporte para **cambiar entre múltiples estilos con un solo clic**.

## I. Canalización

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

## II. Correspondencia de palabras de activación

| Agente | Responsabilidad en una línea | Palabras de activación (lenguaje natural, incluidas las variantes sinónimas) |
|--------|-----------|------------------------------|
| **orchestrator** (Cerebro) | Recibe un requisito → entrega el estilo seleccionado + un paquete de tareas autocontenido → consolida la entrega | Nuevo: `开始策划` `新建文档` `帮我做一份PPT` `帮我写一份Word` `帮我做一份Excel` `做文档`; Editar: `编辑文档` `修改文档` `改这份PPT` `改这个Word` (prefiere la rama de edición cuando el usuario proporciona un archivo existente) |
| **word-crafter** | Recibe plan + estilo → genera un .docx conforme a la especificación | Nuevo: `撰写Word` `写正文` `生成Word`; Editar: `改Word` `修改文档` `编辑这份Word` |
| **slide-designer** | Recibe plan + estilo → genera un .pptx narrativo | Nuevo: `制作PPT` `做幻灯片` `生成PPT`; Editar: `改PPT` `修改这份PPT` |
| **sheet-analyst** | Recibe datos + plan + estilo → genera un .xlsx orientado a conclusiones | Nuevo: `制作Excel` `做报表` `生成Excel` `分析数据`; Editar: `改Excel` `修改表格` `编辑这份Excel` |

**Reglas de activación**: las tareas nuevas (sin archivo de plan correspondiente) siempre entran primero en `orchestrator` para su clarificación; una vez que `plan-NN.md` es confirmado por el usuario, usa las palabras de activación dedicadas de cada experto para ir directamente. Las palabras de activación no entran en conflicto entre sí ni con los comandos integrados del sistema.

## III. Estructura de directorios

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

## IV. Contratos de archivos (salida aguas arriba → lecturas aguas abajo)

1. **Planificación → cada experto (nuevo)**: `outputs/plans/plan-NN.md`, con un formato de metadatos de encabezado fijo:
   ```yaml
   ---
   doc_id: plan-NN
   title: <título>
   doc_type: word | ppt | excel
   style_id: <consulta shared/style-catalog.md — 14 estilos: 6 clásicos + 8 inspirados en la comunidad; selector en shared/style-switcher.md>
   brand_kit: <opcional; escribe brand-NN cuando el usuario proporcione activos de marca, reglas en shared/brand-kit.md; omítelo si no los hay>
   audience: <audiencia>
   purpose: <propósito>
   length: <longitud>
   data_source: <opcional, ruta del archivo de datos Excel>
   ---
   ```
1b. **Tarea de edición → cada experto (editar)**: `outputs/plans/edit-NN.md`:
   ```yaml
   ---
   doc_id: edit-NN
   source_file: <ruta del documento original, proporcionado/arrastrado por el usuario>
   doc_type: word | ppt | excel
   mode: edit
   change_request: <solicitud de cambio, específica por párrafo/página/celda>
   keep: <elementos que deben conservarse: formato original / conclusiones existentes / datos>
   style_id: <por defecto, el estilo original; cambiar de estilo requiere especificación explícita del usuario>
   ---
   ```
2. **Salida de cada experto**: `outputs/<type>/<filename>`, acompañada de un `-meta.md` con el mismo nombre (que registra: doc_id, style_id, **brand_kit (si lo hay)**, fecha de generación, fuente de datos, notas de cambios).
3. **Iteración**: todos los comentarios de revisión del usuario pasan por `send_message` de DSH para continuar la conversación del agente atómico y revisar, preservando el contexto; las revisiones deben anotarse en las "notas de cambios" de `-meta.md`, nunca reescribir silenciosamente el contenido confirmado.
4. **Kit de marca (v2)**: cuando el usuario proporciona activos de marca (colores de marca / Logo / fuentes / plantillas) → orchestrator los recopila en `shared/brand-kit.md`, el plan escribe `brand_kit: brand-NN` y los tres expertos los aplican de manera uniforme (la marca tiene prioridad sobre el catálogo de estilos); al entregar varios documentos bajo el mismo brand_id, orchestrator verifica la consistencia de marca (consulta `shared/brand-kit.md`).

## V. Créditos de la comunidad (referencias de diseño)

| Fuente | Puntos tomados |
|------|--------|
| [anthropics/skills](https://github.com/anthropics/skills) (oficial) | El paradigma de docx/pptx/xlsx "leer archivo existente → generar → verificar que se pueda abrir" |
| [johnson7788/skill-ppt-agents](https://github.com/johnson7788/skill-ppt-agents) + [MultiAgentPPT](https://github.com/johnson7788/MultiAgentPPT) | PPT "esquema → contenido → diseño" en varias etapas, con aceptación independiente en cada etapa |
| [rafalozan0/DocFlow](https://github.com/rafalozan0/DocFlow-Presentations-and-Docs-Skill) | python-pptx/docx/openpyxl unificados para el trío + directorio de plantillas basado en archivos (cambio de múltiples estilos) |
| [cabbage2000-lab/data-analysis-skills](https://github.com/cabbage2000-lab/data-analysis-skills) | Estructura de seis párrafos de Excel con conclusiones primero, datos trazables, sin fabricación, correlación ≠ causalidad |
| [fleurytian/awesome-claude-skills](https://github.com/fleurytian/awesome-claude-skills) (ex-McKinsey) | Estructura narrativa del Principio de la Pirámide / MECE |
| [hugohe3/ppt-master](https://github.com/hugohe3/ppt-master) (repositorio de especificaciones de estilo) | El estilo Cuadrícula suiza (Swiss Grid) y otras especificaciones de diseño → almacenado como el estilo `swiss-grid` |
| [corazzon/pptx-design-styles](https://github.com/corazzon/pptx-design-styles) / [sunchaokun/PPT-Design-Skill](https://github.com/sunchaokun/PPT-Design-Skill) / [GordenSun/GordenPPTSkill](https://github.com/GordenSun/GordenPPTSkill) | Investigación sobre sistemas de estilo de PPT de la comunidad y práctica de plantillas (solo se toman las especificaciones de estilo, no se copian las plantillas) |
| [Tendencias de diseño SlideRabbit 2025](https://sliderabbit.com/blog/inspiring-2025-presentation-design-trends/) y otros | Tendencias de brutalismo / Y2K / degradados → almacenadas como `brutalist` / `retro-y2k` / `aurora-gradient` |
| [Storyset](https://storyset.com/) / [unDraw](https://undraw.co/) / [manypixels](https://www.manypixels.co/gallery) | Ecosistema gratuito de ilustración plana → estilo `flat-illustration` y referencia de mapeo para generación de imágenes |

Todas las fuentes anteriores superaron la revisión de seguridad y estado (inyección de prompts / instrucciones maliciosas / exfiltración de datos / licencias de derechos de autor / estado activo, 2026-08-24); solo se extrajo la esencia, se adaptó y se reescribió en la biblioteca, sin copia literal; todas las entradas del catálogo de estilos citan los enlaces de origen.

## V·5 Reutilización del plano

Plano existente: **办公文档** (`blueprints/办公文档.md`, que contiene el diagrama de topología / la lista de agentes / los registros de decisión ADR / las partes reutilizables de la comunidad / las lecciones aprendidas). Para necesidades similares (flujos de trabajo multiagente de tipo documento), puedes decir 「**复用 办公文档 拓扑**」 — solo aclara las diferencias, sin rediseñar; los cambios iterativos se escriben de vuelta tanto en el plano como en este documento.

## VI. Instrucciones de primer uso

1. Tarea nueva: solo di 「**帮我做一份[PPT/Word/Excel]**」 o 「**开始策划**」, y orchestrator preguntará sobre tema / audiencia / extensión / estilo, **y preguntará si hay activos de marca (si los hay, produce de manera uniforme según el kit de marca)**.
2. Elige el estilo: orchestrator primero pide una dirección (profesional-estable / moderno-premium / personalidad-distintiva / contenido-amigable) y luego ofrece 3–4 opciones de estilo concretas (14 estilos: Profesional de negocios / McKinsey / Académico / Minimalista / Creativo / Oficial + Cuadrícula suiza / Glassmorphism / Editorial de revista / Tecnología oscura / Degradado aurora / Ilustración plana / Brutalista / Retro Y2K), con enlaces a fuentes de la comunidad como referencia.
3. Tras confirmar el plan, di la palabra de activación del experto correspondiente (p. ej., 「制作PPT」) para producir el documento.
4. Modificar (un documento ya entregado): solo di qué cambiar, y orchestrator usará `send_message` para que el agente atómico continúe la conversación y haga la revisión.
5. **¿No te convence el estilo? Cambia una ronda**: solo di 「换一种风格/风格不满意/换个风格重做」 — el contenido y la estructura se conservan por completo, orchestrator vuelve a ofrecer opciones de estilo, produce un nuevo plan con un nuevo `style_id` y lo rehace; el archivo nuevo no sobrescribe el anterior, y la entrega incluye una comparación de estilo antes/después. Se recomiendan como máximo 3 rondas consecutivas y luego pasar al ajuste fino de detalles o proporcionar una imagen de referencia.
6. **Editar un documento existente**: envíame la ruta del .docx/.pptx/.xlsx (o arrástralo) y di 「**改这个Word / 改这份PPT / 修改表格** + qué cambiar」, orchestrator primero identifica la estructura/estilo del documento original, luego hace que el experto realice cambios incrementales y la entrega incluye una lista de cambios detallada sin sobrescribir el archivo original.

## VII. Control de seguridad

**Control de seguridad superado (2026-08-23)**: todos los archivos AGENT.md y knowledge/ superaron cinco revisiones — inyección de prompts / instrucciones maliciosas / exfiltración de datos / envenenamiento de la cadena de suministro / seguridad de la plataforma; el contenido de origen comunitario es trazable, con las conclusiones de revisión adjuntas.
**Re-revisión del control de seguridad superada (2026-08-24, tras la iteración v2 del kit de marca)**: el recién añadido `shared/brand-kit.md` y las secciones del kit de marca de cada AGENT.md superaron la re-revisión (sin inyección de prompts / instrucciones maliciosas / exfiltración de datos / residuos de credenciales), y el registro de palabras de activación quedó congelado sin cambios.

## VIII. Mejoras de capacidades de PPT v3 (2026, destiladas tras investigar 7 complementos de PPT de la comunidad)

> Tomando los **mecanismos** (no el código) de dsh-ppt / PPTKit Presentation / @yejiming/dsh-ppt / pptfast / pptwise / DeepSeek Design / dsh-univer-office, slide-designer añade cuatro capacidades:

- **Control de calidad de revisión visual**: `agents/slide-designer/scripts/visual-qa.py` detecta automáticamente desbordamiento de texto / superposición de elementos / contraste insuficiente / elementos fuera de límites / fuentes faltantes (`python visual-qa.py <deck.pptx>`); `knowledge/visual-qa.md` proporciona una lista de verificación de inspección manual y reglas de corrección automática.
- **Colaboración con el ecosistema de complementos**: `knowledge/plugin-ecosystem.md` registra los comandos de instalación y los modos de colaboración de 7 complementos de PPT de la comunidad (esta skill produce el primer borrador narrativo → los complementos se encargan del refinamiento / la revisión de la visualización).
- **Mapeo de alias de temas**: `shared/style-catalog.md` añade mapeos de nombres de temas de la comunidad a `style_id` (Data Drift→`dark-tech`, Swiss Pulse→`swiss-grid`, Velvet Standard→`editorial-magazine`, etc.).
- **Mejoras**: especificación primero (las especificaciones a nivel de página se finalizan antes del renderizado), artefactos duales de vista previa en HTML, extracción de colores/fuentes de los PPT existentes de la empresa al kit de marca.
