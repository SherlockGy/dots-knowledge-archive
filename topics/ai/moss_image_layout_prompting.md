# MOSS-Image 布局提示词：JSON、坐标与 Prompt Enhancer

归档与最后核查：2026-11-04。适用范围：OpenMOSS/MOSS-Image 官方公开的 Layout 输入约定与 Prompt Enhancer skill。仓库证据固定于提交 `1961ca3d111b828e65278656cde0c8f94484f4dd`；技术报告采用 arXiv v1。本文核查了文档、编译脚本和测试，没有进行图像生成效果评测。

## 摘要

MOSS-Image 的 Layout prompt，是把整体画面、每个元素的外观、可见文字及位置写成 JSON，再作为**文本提示词**交给模型。它适合海报、封面、知识图和需要控制相对位置的多元素构图。

最容易弄混的是两个阶段：

- **给人或 Agent 编辑的草稿**：左上角为原点，绝对像素，`abs` + `xyxy`，包含 `width`、`height`。
- **当前官方工作流输出给模型的 Layout**：画布中心为原点，绝对像素，`abs_centralized` + `yxyx`；宽高在生成参数中另设。

推荐做法是先确定画布，写易读草稿，用官方编译器校验和转换，然后以相同尺寸生成。坐标能表达布局意图，但不能代替文字校对与成图检查。官方用户指南仍把 Layout Prompts 标为 Experimental。[官方用户指南][guide] · [官方 skill][skill]

## 1. 三层内容各自解决什么问题

| 层次 | 放什么 | 回答什么问题 |
| --- | --- | --- |
| `summary.caption` | 画面主题、媒介、整体风格、光线、配色与构图意图 | 整张图是什么样 |
| `items[].caption` | 单个物体、人物或文字的外观与处理方式 | 这一块画成什么样 |
| `items[].bbox` 与 `text` | 元素的位置范围、需要逐字呈现的内容 | 放在哪里，具体写什么 |

例如“上方深蓝色大标题，下方一座山”只用自然语言也能表达。Layout 进一步把标题和山拆成两个元素，并分别给出框的位置，减少模型自行猜测构图的空间。[官方 README][readme]

它通过模型的文本条件通路发挥作用。工程上应把 bbox 看作**布局引导**，不应当作浏览器排版、矢量绘图或逐像素裁切指令；JSON 合法也不能保证文字、字体与位置完全正确。[技术报告 §2][paper-interface]

### 什么时候不必用 Layout

- 单主体照片、普通插画，且位置无需细控：优先用完整自然语言 caption。
- 有指定文字位置、多个信息区、明确空间关系：Layout 更值得使用。
- 必须严格符合品牌字体、尺寸、对齐与完整文案：可让模型生成视觉素材，再用确定性的排版工具完成文字与版式。这是交付建议，不是模型保证。

细节多不等于必须使用 JSON；官方 enhancer 同样区分 caption 与 Layout 两种输出。[官方 skill][skill]

## 2. 当前公开格式的字段

以下以官方 enhancer 的可编辑草稿和编译器为准，避免混入论文训练格式或第三方编辑器字段。

| 字段 | 类型及要求 | 用法 |
| --- | --- | --- |
| `width`、`height` | 正整数，均为 16 的倍数 | 草稿画布尺寸；编译后移除，生成时另设相同值 |
| `bbox_coord_space` | 草稿为 `"abs"` | 编译后变为 `"abs_centralized"` |
| `bbox_format` | 草稿为 `"xyxy"` | 编译后变为 `"yxyx"` |
| `summary.caption` | 非空字符串 | 整体画面描述 |
| `summary.meta` | 可选字符串数组 | 如媒介、图像类型 |
| `summary.canvas` | 可选：`single`、`multi_panel`、`collage` | 构图类别，不是宽高元数据 |
| `summary.palette` | 可选字符串数组 | 整体配色提示 |
| `items` | 非空、扁平数组 | 每个条目是一个需要表达的视觉单元 |
| `items[].label` | `object`、`character`、`text` | 当前编译器接受的元素类型 |
| `items[].caption` | 非空字符串 | 元素外观；文字项可描述字形、颜色、字号层级和表面效果 |
| `items[].bbox` | 四个有限数值 | 草稿中的全画布像素边界，必须在画布内且有正面积 |
| `items[].text` | `text` 类型必须有，且非空 | 画面上要出现的原文；保留语言、标点、大小写和换行 |
| `items[].tags`、`palette` | 可选字符串数组 | 只有确实提供视觉信息时才添加 |

上述可选字段不是越多越好。`summary.caption` 管全局，元素 caption 管局部，避免互相矛盾。[字段说明][layout] · [编译器源码][compiler]

当前编译器不接受 `children`、`parent`、`id`、`font_size` 等额外字段，也不接受嵌套元素树。文字写在杯子上时，文字框仍使用**全画布坐标**，不使用相对杯子的局部坐标。需要文字与主体一起移动时，编辑草稿时同步移动相关框。[字段说明][layout]

这些限制描述的是当前公开工具的输入契约，不能外推成“模型只见过这一种 JSON”。论文讨论了更多训练 schema 和坐标表示；论文中的归一化格式、scene/element 字段与当前 skill 草稿不能直接混拼。[技术报告附录 C.3、F.2][paper-schemas]

## 3. 坐标：顺序与原点必须同时确认

### 3.1 编辑草稿：`abs` + `xyxy`

画布宽 W、高 H，左上角为 `(0, 0)`，x 向右增大，y 向下增大。

```text
bbox = [x_min, y_min, x_max, y_max]

0 <= x_min < x_max <= W
0 <= y_min < y_max <= H
```

这些数值是像素边界，不是百分比，也不是 0–1 或 0–1000 的归一化数值。草稿中不要提前减去中心偏移；交给编译器统一转换。[字段说明][layout]

### 3.2 模型输入：`abs_centralized` + `yxyx`

原点移到画布中心，x 仍向右、y 仍向下。顺序改为先 y 后 x：

```text
bbox = [y_min_centered, x_min_centered, y_max_centered, x_max_centered]

x 范围：-W/2 到 +W/2
y 范围：-H/2 到 +H/2
```

整数像素草稿的转换为：

```text
[x1, y1, x2, y2]
          ↓
[y1 - H/2, x1 - W/2, y2 - H/2, x2 - W/2]
```

反向读回普通像素框：

```text
[a, b, c, d]（中心原点、yxyx）
          ↓
[b + W/2, a + H/2, d + W/2, c + H/2]（左上角原点、xyxy）
```

负数是正常的：x 为负表示在中心左边，y 为负表示在中心上方。`abs_centralized` 里的 `abs` 表示绝对像素尺度，不是把数值取绝对值，也不是中心点加宽高的 `cxcywh` 格式。[官方 README][readme]

### 3.3 具体例子：1024 × 1024 的标题框

官方例子中的最终框为：

```json
[-416, -384, -288, 384]
```

每个轴的中心偏移都是 512。读回普通像素坐标：

- 左边：`-384 + 512 = 128`。
- 上边：`-416 + 512 = 96`。
- 右边：`384 + 512 = 896`。
- 下边：`-288 + 512 = 224`。

因此标题占据 `x=128..896`、`y=96..224` 的区域，宽 768、高 128，位于画布上部。对应草稿框是 `[128, 96, 896, 224]`。[官方例子][readme]

### 3.4 非正方形与改尺寸

以 `1536 × 1024` 为例，x 的中心偏移为 768，y 为 512，不能两个轴都减 512。全画布最终框为 `[-512, -768, 512, 768]`。

若只是把画布从 W×H 改为 W′×H′，想保持相对位置与占比，需要在**草稿**中同时缩放框：

```text
x′ = x × W′/W
y′ = y × H′/H
```

然后重新编译，并设置新的生成尺寸。只改草稿 `width/height` 或生成参数，不会自动按比例缩放元素框。若长宽比也变了，分别缩放两个轴只能保留相对区域，未必保留良好的构图和文字可读性，通常需要重新排版。[字段说明][layout]

官方编译器允许草稿边界为小数：先消除极小浮点误差，再对左/上取 floor、右/下取 ceil，随后减中心偏移，确保输出是包住原框的整数边界。正常使用不必手算这一过程。[编译器源码][compiler]

## 4. 可复用的最小完整例子

以下为本档案自编示例，采用 1024 × 1024 画布，只安排标题与山形插画。它是语法及坐标示例，不代表已测得的成图效果。

### 4.1 保留这份可编辑草稿：`draft.json`

```json
{
  "width": 1024,
  "height": 1024,
  "bbox_coord_space": "abs",
  "bbox_format": "xyxy",
  "summary": {
    "meta": ["travel poster"],
    "canvas": "single",
    "caption": "A quiet travel poster on warm ivory paper. A deep-blue Chinese title sits above a simplified mountain landscape. Use restrained flat shapes and generous empty space."
  },
  "items": [
    {
      "label": "text",
      "caption": "Bold, clean deep-blue Chinese lettering, centered in a single line without a backing panel.",
      "text": "轻装出发",
      "bbox": [128, 96, 896, 224]
    },
    {
      "label": "object",
      "caption": "A compact cluster of layered blue mountain silhouettes, with one pale winding trail and a small warm-orange sun above the ridge.",
      "bbox": [224, 320, 800, 896]
    }
  ]
}
```

描述可以用英文；画面上的中文原文留在 `text`。如果需要两行，写成如 `"轻装出发\n去看远山"`，并给文字框足够高度。JSON 中的 `\n` 在解码后表示换行，不是让图中画出反斜杠和字母 n。[官方 skill][skill]

### 4.2 用官方工具编译

在已取得的官方 MOSS-Image 仓库根目录中执行，草稿与输出分开保存：

```bash
python3 skills/moss-image-prompt-enhancer/scripts/compile_prompt.py \
  draft.json --output prompt.txt
```

编译器负责字段、尺寸和几何检查，转换坐标，删除草稿的 `width/height`，输出一行紧凑 JSON。下面是本示例经该版本官方编译器得到的内容，**生成尺寸仍需另设为 1024 × 1024**：

```json
{"summary":{"meta":["travel poster"],"canvas":"single","caption":"A quiet travel poster on warm ivory paper. A deep-blue Chinese title sits above a simplified mountain landscape. Use restrained flat shapes and generous empty space."},"items":[{"label":"text","caption":"Bold, clean deep-blue Chinese lettering, centered in a single line without a backing panel.","text":"轻装出发","bbox":[-416,-384,-288,384]},{"label":"object","caption":"A compact cluster of layered blue mountain silhouettes, with one pale winding trail and a small warm-orange sun above the ridge.","bbox":[-192,-288,384,288]}],"bbox_coord_space":"abs_centralized","bbox_format":"yxyx"}
```

一行紧凑 JSON 是当前 skill 的交付约定；官方 README 同样用多行 JSON 讲解格式。不要据此把“JSON 有换行”误诊为模型一定不能接收。[官方 skill][skill] · [官方 README][readme]

### 4.3 送入生成入口

- **Gradio**：把编译结果放入 Prompt 框，设置相同宽高。
- **Python / Diffusers**：把 Layout 序列化为字符串传给 `prompt`，另传 `width`、`height`。如果读取的是已编译的 `prompt.txt`，它本身已是字符串，无须再 `json.dumps` 一次。
- **托管 API**：请求体的 `prompt` 值是序列化后的 Layout **字符串**，`size` 另设为如 `"1024x1024"`；不是把 Layout 对象直接塞进 `prompt`。[官方 API 文档][api]

完整 JSON 请求体仍需要由 HTTP 客户端正常序列化。这和“不要把已编译的 prompt 字符串再次单独 JSON 编码”是两回事。

## 5. Prompt Enhancer 做什么，哪些事不做

当前公开 skill 的基本过程是：

```text
用户意图
  → Agent 选 caption 或 Layout
  → 具体化内容、风格、文字和构图
  → 写本地 JSON 草稿
  → Python 编译器校验与序列化
  → 可粘贴提示词
  → 另行交给 MOSS-Image 生成
```

职责分工很明确：Agent 负责内容与构图判断，Python 负责坐标运算和格式。编译脚本本身不调用图像模型，也不负责检查事实、审美或成图质量。普通 caption 草稿只有 `{"caption":"..."}`，编译结果是纯文本；Layout 才产生带框 JSON。[官方 skill][skill]

需要保留的创作约束包括主体、数量、相互关系、画风、排除项和可见原文。不能为了凑模板擅自增加文案，也不能为了装进框而静默删掉内容。事实性知识图还需单独核对知识内容。[官方 skill][skill]

这里描述的是**当前公开的 skill 与编译器**。论文中的 enhancer 方案与 schema 有不同设计；使用具体工具时，应按该工具对应版本的说明输入，而不是把几份格式合成一个“万能 JSON”。[技术报告附录 F.2][paper-enhancer]

## 6. 构图与文字的实用规则

1. **先确定成图尺寸。** 绝对像素坐标依赖尺寸；正文少、标题大和信息密集的版式，需要不同空间分配。
2. **把框分给有意义的视觉单元。** 只有独立摆放有必要时才拆项，不必每个词、每个装饰都单独成框。
3. **文字原文放 `text`，文字样式放 `caption`。** 颜色、字重、字体气质和字号层级用具体描述表达，不向当前编译器添加它不支持的 CSS 式字段。
4. **明确换行与阅读顺序。** 一块两行文字可以共用一个框；两处必须独立定位的文字应分项。
5. **允许有意的重叠和遮挡。** 框不是必须露出的边框；如果希望卡片底色或描边，需在 caption 里描述实际要画出的外观。
6. **让全局、局部和数字一致。** 不要在 caption 写“左侧”，bbox 却落在右侧。重叠关系也要写清楚，不能只指望数组顺序充当图层控制。
7. **在目标像素下检查可读性。** 长段落挤进窄框时，应扩框、重排或调整内容方案，不要只增加“清晰”“高质量”等词。
8. **每次改画布或几何都重新编译。** 保留草稿，修改草稿，不把最终中心坐标 JSON 当作主要编辑文件。

前五项与官方 skill 的构图建议一致；对齐一致性、阅读检查及编辑流程是本档案据此整理的实践建议。[官方 skill][skill] · [字段说明][layout]

## 7. 常见问题与定位顺序

| 现象 | 优先检查 |
| --- | --- |
| 整体偏移、上下左右混乱 | 是否混用 `xyxy` / `yxyx`，是否忘记或重复减中心偏移 |
| 方图正常，横图或竖图异常 | x 是否用 W/2、y 是否用 H/2；是否错把两者都当 512 |
| 换分辨率后构图比例变了 | 草稿框是否随新尺寸缩放或重排，生成尺寸是否与草稿一致 |
| 标题位置对，但文字错、缺字或换行不对 | `text` 是否原样保留，是否需要显式 `\n`，框是否容得下文字；之后仍须检查实际成图 |
| 不想要的卡片、方框出现 | caption 是否暗示了面板、贴纸或卡片；bbox 本身只应标注位置 |
| 编译器报 unknown fields | 是否塞入 `font_size`、`children`、`id` 或论文另一种 schema 的字段 |
| 编译器报 out-of-bounds / exceeds canvas | 草稿是否存在负坐标、越界或已被提前转换成中心坐标 |
| 编译器报宽高无效 | 是否为正整数且为 16 的倍数，不能用浮点数或字符串代替 |
| 编译成功但成图仍偏离意图 | 校验器仅检查结构和数值；再检查描述冲突、框大小、模型版本及实际输出 |

官方编译器还会拒绝零面积/反向框、非有限数值、重复 JSON 键和缺失的必需文字。它不会自动截断越界框，也不会悄悄丢弃未知字段。[编译器源码][compiler] · [官方测试][tests]

## 8. 边界、版本与待验证项

- **Experimental 的含义**：格式可以使用，效果仍应实测。本文没有给出逐像素定位、指定字体、长文零错字或任意分辨率的成功率承诺。
- **当前推荐格式与模型能力范围不同**：请按当前文档使用中心像素 `yxyx`，但不要声称模型完全不能处理归一化坐标。论文记录了不同坐标形式；能否在某个推理实现中稳定复现，需单独测试。[技术报告附录 C.3][paper-schemas]
- **改 Layout 再生成与原图编辑不同**：论文的布局修改示例通过重新文生图完成，不以原图为条件。不要把它理解为原图其他像素必然保持不变。[技术报告 §6][paper-edit]
- **不把示例泛化成性能排名**：本文只核查接口、坐标和工具行为，不据官方展示图推断模型在所有海报、中文长文或复杂构图中都更好。
- **后续更新触发条件**：官方 schema、编译器、模型或主要推理入口改变时，重新核对字段、尺寸规则和编译结果；保留旧版本边界。

## 9. 本次验证与来源

本次已验证：

- 官方编译器自带 8 项单元测试全部通过。
- 本文完整草稿可由该编译器输出上面的最终 JSON；中文原文、两个框和画布约定一致。
- 1024 方图的正反向换算、1536 × 1024 全画布框与等比例缩放示例经程序核对。
- 档案相对链接、README 索引与 WORKFLOW 目录树同步检查。

尚未验证：实际模型成图、特定字体的还原、长文字正确率及不同前端的 UI 操作。语法与坐标测试通过不等同于成图效果通过。

### 来源层级

1. 当前官方仓库文档与源码：用于确认公开接口、字段、转换、校验和交付流程。固定提交为 `1961ca3d111b828e65278656cde0c8f94484f4dd`，避免日后 `main` 更新后找不到当时依据。
2. 官方 API 文档：用于确认托管接口的 `prompt` 字符串与外部 `size` 参数；网页核查于 2026-11-04，未固定版本。
3. 官方技术报告 v1：用于理解条件通路、历史 schema 和能力边界。论文方案与当前公开 skill 分开记录。

### 版本记录

- 2026-11-04：首次归档。补齐两个坐标阶段、字段说明、双向公式、非正方形与尺寸变更、可编译示例、enhancer 工作流、排错和实验性边界。

## 说人话

先告诉模型“整张图长什么样”，再逐项写“画什么、写什么、放哪儿”。编辑时用左上角起算的普通像素框，让脚本转换成模型当前推荐的中心坐标；生成时保持相同宽高。框能帮你管布局，最终文字和画面仍要验收。

[readme]: https://github.com/OpenMOSS/MOSS-Image/blob/1961ca3d111b828e65278656cde0c8f94484f4dd/README.md#layout-generation
[guide]: https://github.com/OpenMOSS/MOSS-Image/blob/1961ca3d111b828e65278656cde0c8f94484f4dd/USERGUIDE.md#layout-prompts-experimental
[skill]: https://github.com/OpenMOSS/MOSS-Image/blob/1961ca3d111b828e65278656cde0c8f94484f4dd/skills/moss-image-prompt-enhancer/SKILL.md
[layout]: https://github.com/OpenMOSS/MOSS-Image/blob/1961ca3d111b828e65278656cde0c8f94484f4dd/skills/moss-image-prompt-enhancer/references/layout.md
[compiler]: https://github.com/OpenMOSS/MOSS-Image/blob/1961ca3d111b828e65278656cde0c8f94484f4dd/skills/moss-image-prompt-enhancer/scripts/compile_prompt.py
[tests]: https://github.com/OpenMOSS/MOSS-Image/blob/1961ca3d111b828e65278656cde0c8f94484f4dd/skills/moss-image-prompt-enhancer/scripts/test_compile_prompt.py
[api]: https://platform.moss.cn/en/docs/reference/layout-image-generation/
[paper-interface]: https://arxiv.org/html/2610.23374v1#S2
[paper-schemas]: https://arxiv.org/html/2610.23374v1#A3.SS3
[paper-enhancer]: https://arxiv.org/html/2610.23374v1#A6.SS2
[paper-edit]: https://arxiv.org/html/2610.23374v1#S6
