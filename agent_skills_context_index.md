# Agent Skills 加载与淘汰策略参考索引

整理日期：2026-10-24
研究基线：2026-10-21 最终深挖答复及后续纠正；ZCode 源码快照为 2026-10-20。
用途：以后讨论 Agent 的 Skills 发现、加载、上下文保留、挤出或安装大量 Skills 时，先查本索引，再根据实际版本补充更新。

## 先读结论

装 100 个 Skill，首先影响的是模型能否看见、理解并选中它们。已读正文能否保留，取决于整个会话的内容量与上下文管理：哪怕只读过 1 个 Skill，后续代码、搜索结果等把历史撑大，也可能使它退出原文输入。安装数量、初始目录预算和运行中正文保留是不同问题。

五者的历史研究均未建立“每个 Skill 有一个永久常驻槽位”的保证。目录被过滤或截短，不等于运行中按 LRU 淘汰正文；文件还在磁盘，也不等于正文仍在当前模型请求中。

## 证据状态与版本边界

本资料归档此前已经完成并发给用户的研究结论，不是 2026-10-24 对全部仓库重新审计。以 2026-10-21 14:10 UTC 最终深挖答复为优先基线，目录数值等未被推翻的细节来自 13:57 UTC 初步答复，并纳入 18:58 UTC 的 100+ Skills 说明与随后的纠正；本次没有重新执行源码测试，也没有把旧结论升级为最新版本承诺。

| 项目 | 历史研究记录的版本 | 证据定位 |
| --- | --- | --- |
| Codex | CLI v0.163.0 | rust-v0.163.0；最终答复引用 83086188ae0614bd882818ff1aa23f3194962012 |
| OpenCode | v1.19.3 | e3c4eb242585dcd77be1a867391b7c25fd47ce52 |
| OpenClaw | v2026.10.5，embedded runtime | 1b58cb9a1b4f4353e1add63c8506db4d890962e8 |
| DeepSeek Harness | v0.2.1-rc.2，deepseek-ai/deepseek-harness | 687f282e8625c6b8d11ff58211a90524be2e4317 |
| ZCode | 2026-10-20 公开源码快照，未记录发布版号 | 3c5b227649371507ddc4eb46ac5133afe6db1cab |

以上 SHA 是原答复引用链接中的标识，未在本次重新验证标签与 commit 的对应关系。不得据此假定托管产品、私有服务端、其他 runtime、开发分支或自定义插件具有相同实现。

## 快速比较

| 项目 | 发现阶段的目录预算 | 运行中的正文处理 | 是否证明逐 Skill LRU 淘汰 |
| --- | --- | --- | --- |
| Codex | 默认模型窗口 2%；先缩描述，再省略条目 | 工具输出限制与会话压缩；不保证原文恢复 | 否；已见 LRU selector 是 shadow 实验 |
| OpenCode | 核查版本未见目录总量裁减 | prune 保护 skill；完整 compaction 不同样保护 | 否 |
| OpenClaw | 默认 150 项且 18,000 字符；可缩描述、减条目，另可搜索 | 普通 toolResult，可受适用的 pruning、截断与 compaction 影响 | 否 |
| DeepSeek Harness | 每条描述默认约 500 字符；未见默认总量淘汰 | 通用输出裁剪、压力裁剪、完整 compaction | 否 |
| ZCode | 20,000 字符是整目录去描述的降级阈值，不是硬上限 | 正文读取截断与完整 compaction；特定工作流检测后重读 | 否 |

结论是：存在，但必须区分“目录被省略”“正文被截断”“旧工具结果被清理”“会话被摘要”四件事。不能笼统地说“Skill 按需加载后会一直留着”，也不能把这些都叫作 LRU 淘汰。

最关键的发现：五者都不能保证已经加载的 Skill 正文永久、完整地留在模型上下文里。OpenCode 对 Skill 有局部保护；ZCode 甚至明确实现了“压缩后视为没加载过，特定工具要求重读”。

此前研究答复记录的证据截至 2026-10-21。Codex 以 CLI v0.163.0、OpenCode 以 v1.19.3、OpenClaw 以 v2026.10.5、DeepSeek Harness 以 v0.2.1-rc.2 为基准；ZCode 核对 10 月 20 日公开源码。自定义插件、其他运行时和开发分支可能不同。

## 三种状态与四类缩减

- 已安装：文件或注册信息在磁盘、仓库或服务里。
- 已发现：模型看到名称、描述等目录信息，知道有这个 Skill。
- 已加载：正文真正进入本次模型请求。

前三者不是一回事；“以前加载过”也不等于“现在的请求还带着正文”。进程缓存、完整会话日志、Prompt Cache，都不能替代检查当前实际送给模型的内容。

## Codex

**目录有明确预算，正文没有永久保留承诺**

加载方式：先提供技能目录。普通本地 Skill 可以由用户显式选择后注入正文，也可以由模型按路径读取 SKILL.md；云端或 executor package 则使用 skills.read，支持分页。不是所有 Codex Skill 都必经同一个专用工具。

目录层确实存在“挤出”：默认 metadata 预算是模型上下文窗口的 2%。放不下时先缩短描述；连名称和路径都放不下时，会省略部分条目。省略只是没有展示在目录里，不是删除 Skill，明确选择或已知路径仍可能加载。

正文层不是“加载新 Skill 就卸载旧 Skill”。但是：
- 通过普通工具读取的正文，可能受工具输出长度限制。
- 部分 package/plugin 注入路径有 8,000 bytes 截断，不能泛化成所有本地 Skill 都有这个上限。
- 在 remote compaction v2 中，显式注入的 Skill 正文虽然使用 user role，仍被识别为上下文片段，不享受“真实用户消息原文保留”的待遇；普通读取结果也没有逐字保留保证。
- 压缩后重建的是技能目录等基础上下文，没有发现把所有已用 Skill 正文自动重新灌回的机制。服务端压缩表示可能保留部分含义，但不能据此保证原文完整。
- 最终深挖还覆盖远端压缩、本地摘要与 effective 模型配置 TokenBudget reset（换窗口）三种路径，不能只拿 remote compaction v2 代表全部路径。远端压缩表示不透明，客户端缺少原文保留保证，不等于内容完全忘掉。

还有一个容易误读源码的点：确实存在 LRU selector，但它是 shadow 实验，不改变模型实际看到的目录，不能说“Codex 已上线 Skill 正文 LRU 淘汰”。

最终深挖依据：[压缩路由与实现](https://github.com/openai/codex/blob/83086188ae0614bd882818ff1aa23f3194962012/codex-rs/core/src/session/turn.rs#L1507-L1613)、[正文保留筛选](https://github.com/openai/codex/blob/83086188ae0614bd882818ff1aa23f3194962012/codex-rs/core/src/compact_remote_v2.rs#L480-L578)。

目录等补充依据：[目录预算](https://github.com/openai/codex/blob/rust-v0.163.0/codex-rs/ext/skills/src/render.rs)、[压缩保留规则](https://github.com/openai/codex/blob/rust-v0.163.0/codex-rs/core/src/compact_remote_v2.rs#L480-L578)、[LRU 实验定位](https://github.com/openai/codex/blob/rust-v0.163.0/codex-rs/ext/skills/src/dynamic_skill_selector.rs#L40-L58)。

## OpenCode

**保护 Skill 不被 prune，但不保护它跨完整 compaction**

加载方式：目录进入模型上下文；调用 skill 工具后，正文作为工具结果进入历史，参考资料只提供路径，不全部展开。初次工具返回仍受通用输出限制，默认 2,000 行或 50 KiB，超出部分保存到文件供后续读取。

这里有一个非常重要的区别：

- 旧工具结果清理 prune：源码明确把 skill 放进保护名单，跳过这类结果。该发布版的 prune 默认关闭，开启后仍有此豁免。
- 完整会话压缩 compaction：没有相同保护。落入待摘要旧历史的工具结果，送给摘要模型前统一截到最多 2,000 字符，skill 也不例外；随后旧历史由摘要替代。落在近期保留区的正文可以暂时留下。

加载入口也重要：用户通过 /skill-name 直接展开成用户消息，不走 agent skill 工具的 50 KiB 初次输出截断，也不走工具结果的 2,000 字符摘要输入截断，但仍可能被摘要化；普通 read 读取同一份 SKILL.md，则不继承 skill 工具的 prune 豁免。[手动技能命令展开](https://github.com/anomalyco/opencode/blob/e3c4eb242585dcd77be1a867391b7c25fd47ce52/packages/opencode/src/command/index.ts#L139-L159)。

因此，“OpenCode 不会清理 Skill”这个说法只对第一条路径成立。完整压缩后，目录可以再次出现，但旧正文不会因此自动恢复。

这也是原研究中值得注意的细节：不只是摘要可能漏掉内容，长 Skill 的后半部分甚至可能没有进入摘要输入。

依据：[Skill 加载](https://github.com/anomalyco/opencode/blob/e3c4eb242585dcd77be1a867391b7c25fd47ce52/packages/opencode/src/tool/skill.ts)、[prune 豁免与摘要截断实现](https://github.com/anomalyco/opencode/blob/e3c4eb242585dcd77be1a867391b7c25fd47ce52/packages/opencode/src/session/compaction.ts)。

注意：2026-10-21 研究时的 OpenCode v2 开发分支已经重构这部分，不能把 v1 的保护名单直接当作 v2 承诺。

## OpenClaw

**目录会限额，已读正文可被通用清理**

以下指 OpenClaw 自己的 embedded runtime，不是它调用外部 Codex 等运行时的情况。

加载方式：先注入目录，使用时通过 skills_read 或通用 read 读取。发布版已经有 skills_search，未进入初始目录的技能仍可检索。

目录默认最多 150 项、18,000 字符。超限先收缩描述，名称和位置仍放不下时才减少条目。这是目录展示预算，不是按最近使用时间淘汰已读正文。

正文的风险来自另一层：读取结果是普通 toolResult，没有内置 Skill 专属的不可清理标记。
- 配置及路由适用时，cache-TTL pruning 可以把旧工具结果缩成头尾，进一步用占位符清空；技能结果没有默认豁免。
- 部分直连 Anthropic API-key 路由使用服务端工具结果清除，不能仅按本地 TTL 理解。
- 完整 compaction 同样可能把旧正文变成摘要。

最终深挖纠正：普通 aggregate 预算保护已发送的 frozen 内容，不能说“又读了几个 Skill，就会常规挤掉前面已经发送的 Skill”。条件性 pruning/clearing 与普通 aggregate 预算不是同一条路径。[已发送内容保护](https://github.com/openclaw/openclaw/blob/1b58cb9a1b4f4353e1add63c8506db4d890962e8/src/agents/embedded-agent-runner/tool-result-truncation.ts#L1052-L1145)。

不能理解成“读了五分钟就一定删除”：TTL、上下文压力、近期消息保护、provider 和配置共同决定是否触发。

通用 read 路径还有重复读取去重。压缩后会清掉去重状态，允许重新读取；这不等于后台自动重读，更不等于正文一直被固定保留。

依据：[目录限额](https://github.com/openclaw/openclaw/blob/1b58cb9a1b4f4353e1add63c8506db4d890962e8/src/skills/loading/skill-prompt-limits.ts)、[工具结果清理实现](https://github.com/openclaw/openclaw/blob/1b58cb9a1b4f4353e1add63c8506db4d890962e8/src/agents/embedded-agent-runner/tool-result-truncation.ts#L60-L295)、[官方 pruning 说明](https://docs.openclaw.ai/concepts/session-pruning)。

## DeepSeek Harness

**正文会经过通用裁剪，压缩后只明确恢复目录**

加载方式：初始注入名称与描述；调用 skill 后才返回正文。用户显式调用技能也可以把正文作为指令消息注入。这里研究的是 deepseek-ai/deepseek-harness，不是 DeepSeek 模型自身。

有三层需要分开看：
- 超大工具输出：默认超过约 12,500 estimated tokens，保留头尾和完整结果文件路径。
- 上下文压力裁剪：当前工具结果超过默认 8,192 字符时，可缩成前 4,096、后 1,024 字符，中间移出当前输入；没有 skill 豁免。
- 完整 compaction：旧历史改为摘要，近期历史按预算保留；Skill 正文没有永久 pin。

最终深读确认默认 standard 装配挂载 spill / pruner；不是只在未启用的插件中找到代码。用户 /name 直接注入正文是另一条路径：避开上述工具结果裁剪，但不避开历史摘要。不能把 native skill 工具路径的裁剪结论套给所有加载入口。

目录被压缩掉后，工具插件在后续 pre-step 重新发布当前目录，不是所有重试都会立即恢复；没有对应的“自动恢复所有已加载正文”逻辑。原始日志仍然保存，不表示下一次模型请求仍然携带全文。

另一个独有的易错点：PTC/code mode 中，仅执行 await tools.skill(...)，但不把结果 return 或打印出来，可能出现“工具调用成功，正文却从未进入模型上下文”。这不是后来被挤出，而是只读到了程序内部变量，模型没看到。additionalContexts 扩展存在例外，但默认 skill text 路径仍需显式输出。

依据：[Skill 加载与目录恢复](https://github.com/deepseek-ai/deepseek-harness/blob/687f282e8625c6b8d11ff58211a90524be2e4317/packages/skill/tool-skill/src/index.ts)、[正文裁剪实现](https://github.com/deepseek-ai/deepseek-harness/blob/687f282e8625c6b8d11ff58211a90524be2e4317/packages/compaction/compaction-tool-result-pruner/src/index.ts)、[PTC 可见性规则](https://github.com/deepseek-ai/deepseek-harness/blob/687f282e8625c6b8d11ff58211a90524be2e4317/packages/core/tools/README.md#ptc-mode)。

## ZCode

**明确承认压缩会挤出正文，并对特定工作流强制重读**

加载方式：目录放进请求前缀，Skill 工具按需返回正文。

目录默认 20,000 字符预算。超出时整份目录降级为名称、别名和路径，丢掉描述，而不是逐个淘汰技能。这个数字是降级阈值，不是最终输出硬上限。

正文读取默认最多 100,000 bytes，超出截尾；dynamic-workflows 特例为 200,000 bytes。

完整压缩后保留的是前缀、摘要、部分近期历史等。技能目录属于保留前缀；旧 Skill 调用的正文则可能离开当前上下文。

这里有最直接的代码证据：loaded-skills.ts 明确解释，compaction 把 Skill 调用挤出去后，正文也不在了。因此“是否加载过”检查的是当前模型可见历史，而不是一个永久记录的 session Set。

具体到 dynamic-workflows：如果压缩后当前历史不再含成功加载记录，相关创作工具会拒绝继续，要求先重新加载技能。这是“检测失效后强制重读”，不是所有 Skill 都自动恢复。检查只认当前历史中的成功调用记录，不验证正文是否完整，也不覆盖全部技能。

另有条件性恢复：通过普通 Read 读取的 SKILL.md，有机会被“最近读取内容恢复”机制补回；原生 Skill 工具本身不走这套恢复。所以不能说“所有路径都绝不恢复”。[普通 Read 恢复](https://github.com/zai-org/ZCode/blob/3c5b227649371507ddc4eb46ac5133afe6db1cab/apps/zcode-cli/packages/core/src/runtime/helpers/compact-post-reminders.ts)。

另外，microcompact 需要显式开启，默认可清理工具名单不含 Skill；但它读取的参考资料若来自 Read 等工具，仍可能被清理。不能把 microcompact 的局部保护当作完整压缩保护。

依据：[目录预算](https://github.com/zai-org/ZCode/blob/3c5b227649371507ddc4eb46ac5133afe6db1cab/apps/zcode-cli/packages/core/src/context/sections/skills.ts)、[已加载状态与压缩关系](https://github.com/zai-org/ZCode/blob/3c5b227649371507ddc4eb46ac5133afe6db1cab/apps/zcode-cli/packages/core/src/agent/loaded-skills.ts)、[工作流重读门](https://github.com/zai-org/ZCode/blob/3c5b227649371507ddc4eb46ac5133afe6db1cab/apps/zcode-cli/packages/core/src/tool/handlers/workflow-skill-gate.ts)。

## 已加载正文的判断方法

这些产品大多不是“给每个 Skill 分配一个常驻槽位，新的来了挤掉旧的”，而是“把 Skill 正文放进对话，之后受到对话上下文管理影响”。

真正该问的不是“支不支持 Skills”，而是：
1. 正文第一次是否完整进入了模型请求？
2. 旧工具结果清理是否豁免它？
3. 完整压缩后，是否原文恢复，还是仅剩摘要和目录？
4. 如果已经不在，是否能可靠地检测并重新加载？

这一历史基线中，OpenCode 的特点是第 2 项有局部保护；ZCode 的特定工作流在第 4 项有明确机制。五者都不能仅凭“加载成功”就认为整场长任务会持续、完整地遵守那份 Skill。

## 安装 100 个以上 Skills 的人话结论

前提：这 100 个 Skill 均已启用且可用；仍只适用于上述历史版本。

先说共同点：装了 100 个，不会开场就把 100 份正文全部塞进去。先占空间的是“技能名称和简介”，用到哪个才加载正文。

1. Codex：装得下，但模型默认看到的介绍可能被缩短，甚至有些技能没展示出来。

目录有预算，随模型上下文窗口变化。超预算先缩介绍，再放不下才省略部分条目。因此不是“超过 100 个就出问题”，而是取决于简介、路径有多长，以及模型窗口多大。

人话：它会控制菜单长度；菜还在，但菜单可能写不全。明确点名或按已知路径仍可加载。

2. OpenCode：基本把全部可用技能的简介摆出来，装得越多，菜单越厚。

核查版本没有目录总量裁减。100 个不会因为数量自动被淘汰，但简介本身持续占用上下文，留给实际任务的空间相应减少。

人话：不主动藏技能，也不替你给菜单瘦身。不是把 100 份正文都加载了，而是让模型面对 100 份简介。

3. OpenClaw：初始菜单有限长，但没展示的还能搜索。

默认最多展示 150 项，同时受 18,000 字符预算限制。所以 100 个不一定都能完整介绍：描述、路径长，也会先缩介绍，必要时省略条目。超过 150 个，默认初始目录就不会全部列出。

人话：先给一份限长菜单，没列出来的可以再查；不是卸载了那些技能。

4. DeepSeek Harness：每份介绍会限长，但装得越多，总菜单仍然越长。

每条描述默认最多约 500 字符；这不是全部技能合计 500 字符。已查默认路径没有“装多后只保留常用几个简介”的总量淘汰机制。

人话：100 个技能就是很多份简介，不会自动靠淘汰冷门技能替你节省目录空间。

5. ZCode：装得多，可能突然从“名字＋介绍”变成“只有名字＋路径”。

目录超过约 20,000 字符，会整体去掉描述，不是挑一部分技能删除。100 个技能如果每份介绍约 200 字符，仅介绍就已经接近这个阈值，还没算名称和路径。

人话：技能都还在，但模型失去了“这个技能适合什么时候用”的说明。名字不直观时，自动选择会缺少依据；不是保证它一定选错。

最简单的对比：
- Codex：菜单超预算，缩介绍，再省略条目。
- OpenCode：菜单基本全摆，自己承担上下文开销。
- OpenClaw：限长菜单，遗漏项还能搜索。
- DeepSeek Harness：单条介绍限长，总菜单仍随数量增长。
- ZCode：菜单超预算，整体去掉介绍，留下名字和路径。

至于“装多了会不会把之前用过的 Skill 挤掉”：安装数量本身通常不是触发器。关键是整个会话内容量及具体清理策略，不必加载很多 Skill：哪怕只读过一个，后续代码、搜索结果等使历史增长，也可能触发工具结果清理或会话压缩。

所以：装 100 个，首先影响的是“模型能不能看见、理解并选中它们”；整个会话持续增长，会进一步影响“已经读过的正文还能保留多少”。

## 源码导航与待补字段

每个项目段落已保存固定版本的源码链接。最终深挖答复声明追过启动入口、默认配置、最终请求构造、压缩恢复链路与测试断言，但没有完整列出函数名称，本次不根据文件名猜函数；后续核验时补充精确符号、调用链与测试。

- Codex：codex-rs/ext/skills/src/render.rs（目录预算）；codex-rs/core/src/compact_remote_v2.rs 第 480–578 行（上下文片段保留规则）；codex-rs/ext/skills/src/dynamic_skill_selector.rs 第 40–58 行（shadow 定位）。另看 codex-rs/core/src/session/turn.rs 第 1507–1613 行的压缩路由。函数名待源码复核；8,000 bytes 的具体 package/plugin 分支位置原答复未给出，不能当作通用上限。
- OpenCode：packages/opencode/src/tool/skill.ts（加载入口）；packages/opencode/src/session/compaction.ts（prune 豁免与摘要前截断）。另看 packages/opencode/src/command/index.ts 第 139–159 行的手动命令展开。函数名及通用初次输出截断的准确实现文件待补；v2 必须独立核验。
- OpenClaw：src/skills/loading/skill-prompt-limits.ts（目录）；src/agents/embedded-agent-runner/tool-result-truncation.ts 第 60–295 行（工具结果截断）。同文件第 1052–1145 行定位 frozen 内容保护，不能与旧结果清除混为一谈。历史答复也引用官方 session-pruning 文档。TTL pruner、provider 服务端清除、read 去重与重置的完整调用链和函数名待补，不能只用截断文件证明所有清理路径。
- DeepSeek Harness：packages/skill/tool-skill/src/index.ts（加载与目录恢复）；packages/compaction/compaction-tool-result-pruner/src/index.ts（压力裁剪）；packages/core/tools/README.md 的 ptc-mode（模型可见性）。函数名、500 字符单条描述限制及 12,500 estimated tokens 通用输出阈值的精确实现位置待补。
- ZCode：apps/zcode-cli/packages/core/src/context/sections/skills.ts（目录）；apps/zcode-cli/packages/core/src/agent/loaded-skills.ts（基于当前可见历史判断）；apps/zcode-cli/packages/core/src/tool/handlers/workflow-skill-gate.ts（工作流重读门）。另看 apps/zcode-cli/packages/core/src/runtime/helpers/compact-post-reminders.ts 的普通 Read 条件恢复。函数名、正文读取字节限制和 microcompact 默认名单的精确实现位置待补。

发现/扫描目录来源、同名冲突优先级、平台与权限过滤等细节，原完整答复没有逐项保留，本索引不补猜。这里的“没有发现”仅限当时已查路径，不能证明任意配置、插件或后续版本都不存在其他策略。

## 后续更新方法

1. 先记录具体产品、CLI/runtime、发布版或 commit、provider 与配置，不把产品名等同于单一实现。
2. 分别核验磁盘发现与过滤、模型可见目录、正文注入/工具读取、首次输出截断、旧工具结果清理、完整压缩、重载检测七个阶段。
3. 对每一条数值保留单位和适用条件：字符、bytes、estimated tokens、项目数不是同一量纲。
4. 更新现状时保留这份历史基线，并记录新日期、版本、源码函数、变更结论及证据，不静默覆盖版本边界。
5. 判断“正文仍在”应看当前模型请求或可见历史，不能只看加载成功日志、缓存或已安装清单。
6. 如需验证长期执行效果，再做实际请求/压缩前后实验；源码机制不等于模型一定准确选中或遵守 Skill。

## 资料来源

- 2026-10-21 14:10:16 UTC，本对话“五项目 Skills 加载与淘汰机制”最终深挖答复，为优先结论来源。
- 2026-10-21 13:57:26 UTC，本对话初步答复，仅保留未被最终深挖修正的目录预算、输出上限等补充细节及其链接。
- 2026-10-21 18:58:10 UTC，本对话“装 100 个或者更多 Skills 会怎样”逐项目解释。
- 2026-10-21 18:58:20 UTC，补充纠正：只加载 1 个 Skill 也可能因其他历史增长而失去原文，关键是整个会话内容量。
- 2026-10-24 本次整理只恢复、归并上述内容和证据定位，未重新从零展开研究。

更新记录：2026-10-24 建立知识归档索引；以最终深挖答复替代初步答复作为优先基线，纳入手动注入与工具调用差异、frozen 保护、条件性 Read 恢复、重读门完整性边界及单 Skill 纠正。
