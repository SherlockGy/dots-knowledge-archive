# 视频项目技术索引

更新：2026-10-24。用途：以后讨论现有视频制作方案时，先查本索引；事实变化及时更新，旧状态先复核。按实际成片管线区分素材、绘图引擎、动画编排、声音和导出，不把旧代码或同仓库另一部片子的实现混入。

本次整理依据 2026-10-22、23 日已完成的源码核查与成片比较，及已确认的项目资料；不是重新运行全部项目。未恢复的成片文件名、版本号、音源或规格明确留空，不凭同源关系补全。排除 devteamdrew 在 X 分享、制作方式未确认的视频。

## 已核对的成片仓库

### 三只小猪
- 仓库：SherlockGy/three-little-pigs。核对提交：ae98ac7780dd9f773c55ddde03a7a4dd99c858a7。
- 成片版本：2026-10-22 核查的默认分支对应作品；未恢复独立成片版本号或文件名。
- 引擎：Python + Skia。
- 素材：AI 角色和背景图，处理为透明 PNG。
- 动画驱动：音乐拍点组织镜头；移动、旋转、切换姿势，二维纸偶。
- 音乐 / 配音：音乐驱动已确认；逐项目配音提供商及成片音源未恢复。
- 输出：原始像素帧 → FFmpeg 编码及合音轨。
- 证据（2026-10-22）：https://github.com/SherlockGy/three-little-pigs/blob/ae98ac7780dd9f773c55ddde03a7a4dd99c858a7/src/engine.py#L213-L246

### 小马过河
- 仓库：SherlockGy/little-horse。核对提交：46134bb9451185e942b836e6defc5f1baf7cb744。
- 成片版本：2026-10-22 核查的默认分支对应作品；未恢复独立成片版本号或文件名。
- 引擎：Python + Skia。
- 素材：PNG 纸偶素材；具体素材生成来源未单独恢复。
- 动画驱动：节拍时间轴；与三只小猪同源但已独立修改，不会自动同步。
- 音乐 / 配音：节拍编排已确认；具体配音及音源未逐项恢复。
- 输出：原始像素帧 → FFmpeg。
- 证据（2026-10-22）：https://github.com/SherlockGy/little-horse/blob/46134bb9451185e942b836e6defc5f1baf7cb744/src/engine.py#L121-L226

### 沉没成本
- 仓库：SherlockGy/sunk-cost。核对提交：75dc39c098e9d0d3d6b9d36cdccfe503bd00c3b3。
- 成片版本：2026-10-22 核查的默认分支对应作品；未恢复独立成片版本号或文件名。
- 引擎：Python + Skia。
- 素材：图片、文字和程序图形；具体图片来源未恢复。
- 动画驱动：实际旁白句长与音乐小节共同决定时间轴。
- 音乐 / 配音：有旁白与音乐；具体提供商未逐项恢复。
- 输出：原始像素帧 → FFmpeg。该提交只更换四张封面，不是新视频构建。
- 证据（2026-10-22）：https://github.com/SherlockGy/sunk-cost/blob/75dc39c098e9d0d3d6b9d36cdccfe503bd00c3b3/src/scenes.py#L713-L794

### 间歇性强化
- 仓库：SherlockGy/intermittent。核对提交：3160f4b0b40a33a54fa882d6336b8710b00b6a19。
- 成片版本：2026-10-22 核查的默认分支对应作品；未恢复独立成片版本号或文件名。
- 引擎：Python + Skia。
- 素材：纸张拼贴、半调印刷、撕纸效果；具体原始素材来源未恢复。
- 动画驱动：每秒生成 12 张不同画面，重复输出为 24 fps，刻意制造定格质感。
- 音乐 / 配音：未恢复逐项目音源与旁白细节。
- 输出：原始像素帧 → FFmpeg；24 fps 输出。
- 证据（2026-10-22）：https://github.com/SherlockGy/intermittent/blob/3160f4b0b40a33a54fa882d6336b8710b00b6a19/src/render.py#L34-L70

### 光合作用
- 仓库：SherlockGy/photosynthesis。核对提交：b624d9b05fe8b1326f91f3942606c59083a6ca76。
- 成片版本：2026-10-22 核查的默认分支对应作品；未恢复独立成片版本号或文件名。
- 引擎：Python + Skia。
- 素材：主要用代码画植物、细胞等图形，不靠角色图片表演。
- 动画驱动：多层二维坐标和缩放，形成地球到细胞膜的连续钻入；不是真三维场景。
- 音乐 / 配音：未恢复逐项目音源与旁白细节。
- 输出：原始像素帧 → FFmpeg。
- 证据（2026-10-22）：https://github.com/SherlockGy/photosynthesis/blob/b624d9b05fe8b1326f91f3942606c59083a6ca76/src/scenes.py#L106-L186

### 你好，小生命
- 仓库：SherlockGy/hello-little-life。核对提交：2bb7b08fd98ec6a935216eb4f26290385cb00852。
- 成片版本：2026-10-22 核查的默认分支对应作品；未恢复独立成片版本号或文件名。
- 引擎：JavaScript + Canvas2D。
- 素材：路径、圆、曲线、渐变绘制角色、器官与粒子。
- 动画驱动：先生成旁白并测句长，再编排画面；Playwright 按指定时间出图。
- 音乐 / 配音：旁白驱动；音乐使用 NumPy 波形合成路线。具体旁白提供商未逐项恢复。
- 输出：Playwright / 浏览器出帧 → FFmpeg 编码及合音轨。
- 证据（2026-10-22）：https://github.com/SherlockGy/hello-little-life/blob/2bb7b08fd98ec6a935216eb4f26290385cb00852/src/art.js#L1-L75

### 金陵2500
- 仓库：SherlockGy/nanjing-city-films。核对提交：6c34ffe4b1f44b43a7544a63f5274f14b970e23d。
- 成片版本：2026-10-22 核查的默认分支对应作品；未恢复独立成片版本号或文件名。
- 引擎：JavaScript + Canvas2D，自写三维投影。
- 素材：三维顶点经相机透视与深度排序，画成二维雕版线稿；不是 Three.js。
- 动画驱动：音乐节拍编排；不是单纯移动平面图片。
- 音乐 / 配音：无旁白；南京片共用部分 NumPy 声音合成函数。
- 输出：浏览器出帧 → FFmpeg；位于 history 分支目录，同仓另一部片采用不同引擎。
- 证据（2026-10-22）：https://github.com/SherlockGy/nanjing-city-films/blob/6c34ffe4b1f44b43a7544a63f5274f14b970e23d/history/src/lib.js#L11-L77

### 金陵·一卷未干的水墨
- 仓库：SherlockGy/nanjing-city-films。核对提交：6c34ffe4b1f44b43a7544a63f5274f14b970e23d。
- 成片版本：2026-10-22 核查的默认分支对应作品；未恢复独立成片版本号或文件名。
- 引擎：自写 WebGL2 shader 引擎。
- 素材：照片、水墨图和宣纸纹理；原始图像来源未逐项恢复。
- 动画驱动：shader 计算洇色、墨滴、水波、雾与转场。
- 音乐 / 配音：与金陵2500共用部分声音合成函数；该片旁白情况未单独恢复。
- 输出：浏览器出帧 → FFmpeg；不要套用同仓 history 的 Canvas2D 引擎。
- 证据（2026-10-22）：https://github.com/SherlockGy/nanjing-city-films/blob/6c34ffe4b1f44b43a7544a63f5274f14b970e23d/src/engine.js#L7-L53

### Agent 演进原版
- 仓库：SherlockGy/agent-evolution-film。核对提交：f0c6850a3fbad412f44ae1150d52279f7934bb8c。
- 成片版本：2026-10-22 核查的默认分支对应作品；未恢复独立成片版本号或文件名。
- 引擎：p5.js + p5.brush / WebGL，Canvas2D 叠字与纸纹。
- 素材：代码驱动笔刷绘制角色和场景。
- 动画驱动：线条扰动按 12 Hz 更新；不等于全片只有 12 fps。
- 音乐 / 配音：Agent 片使用 SoundFont 按代码谱曲；原版具体配音状态未单独恢复。
- 输出：Puppeteer 导帧 → FFmpeg；默认 24 fps。
- 证据（2026-10-22）：https://github.com/SherlockGy/agent-evolution-film/blob/f0c6850a3fbad412f44ae1150d52279f7934bb8c/src/core.js#L287-L316

### Agent 演进配音版
- 仓库：SherlockGy/agent-evolution-film-narrated。核对提交：534e30e83dff399f5babeab041615b2d1947b79b。
- 成片版本：2026-10-22 核查的默认分支对应作品；未恢复独立成片版本号或文件名。
- 引擎：p5.js + p5.brush，同源衍生。
- 素材：16 章中 12 章场景文件与原版相同；保留手绘场景。
- 动画驱动：新增旁白时间映射、字幕与混音；旁白过长时暂停或放慢故事时间，但保留呼吸等待机动作。
- 音乐 / 配音：有配音；Agent 片使用 SoundFont 谱曲；具体配音提供商未逐项恢复。
- 输出：浏览器逐帧输出 → FFmpeg 混音编码；不是只给原片贴音轨。
- 证据（2026-10-22）：https://github.com/SherlockGy/agent-evolution-film-narrated/blob/534e30e83dff399f5babeab041615b2d1947b79b/src/chapters.js#L7-L38

### 小 G · 30 秒动效自我介绍
- 仓库：SherlockGy/xiaog-showreel。核对提交：0e7549826fdb9af0f7c500156df87b5851332c96。
- 已核对成片：30 秒、1080p、30 fps；2026-10-23 比较过实际抽帧和渲染代码。
- 引擎：HTML / CSS / SVG + GSAP 时间轴，由 HyperFrames 渲染为视频，不是 Remotion。
- 素材：正片由代码构造黑白橙排版、终端和任务卡片；仓库封面另用 AI 插画，不能算作正片素材来源。
- 动画：DOM / SVG 变换与衔接转场；卡片空间感来自 CSS 透视，不是真三维场景。
- 音乐 / 配音：本次恢复记录未明确具体音源、配音和最终混音命令。
- 输出：HyperFrames 视频渲染；不要自动套用其他仓库的导帧脚本。
- 证据（2026-10-23）：https://github.com/SherlockGy/xiaog-showreel/blob/0e7549826fdb9af0f7c500156df87b5851332c96/compositions/work.html

### 龟兔赛跑 v4
- 仓库：SherlockGy/tortoise-and-hare。成片 v4：236.4 秒、1920×1080、30 fps。
- 实际成片管线：Python + Pillow / NumPy 逐帧合成 → FFmpeg 编码。不是旧 Skia 引擎。
- 素材：AI 生成角色、森林、花草与道具 PNG；白色剪纸边缘、纸艺 / 水彩 / 毛毡纹理由素材带入。
- 动画：四张兔子跑姿循环、质心对齐、身体起伏、前后景视差、统一场景与镜头坐标；丝带图片切条沿曲线排列并模拟断裂垂落。
- 优化：预缩放素材，预生成软阴影，缓存变换图与字幕；主要做图片拼装。
- 音乐 / 配音：使用仓库已提交旁白与音乐；本次恢复记录未确认生成提供商。
- 历史：2026-10-23 检查提交 5d66cef68b6e5b2e4fab56b918ec6b80394b502d 时仍留旧 Skia 代码，但它不是 v4 成片管线。PR #1 于 2026-10-23 01:58:46 UTC 合并，提交 4314c32939073fef52052e270be10b8222381a55，已删除旧 Skia 及专用数据，音乐工具改为可选依赖。
- PR 作者报告最小依赖预处理、三个时间点截图及 12 帧验证通过；这是作者报告，不是本次独立执行结果。
- 重写原因：原始仓库只有打包 v4 的根提交，没有决策日志。简化为缓存拼图管线是代码支持的推断，不是历史动机，也不证明 Pillow 普遍比 Skia 快或好。
- 证据（2026-10-23）：https://github.com/SherlockGy/tortoise-and-hare/blob/5d66cef68b6e5b2e4fab56b918ec6b80394b502d/v2/core.py
- 整理 PR：https://github.com/SherlockGy/tortoise-and-hare/pull/1

## 上传项目：点·线·面
- 来源：用户上传压缩包；包内已有 点线面_v002.mp4。尚未恢复可核对的 GitHub 仓库地址，不按中文名猜 URL。
- 规格：15 秒、1080p、60 fps，128 BPM。
- 引擎 / 素材：Python + Skia；点、线、色块和文字直接由代码绘制，不依靠 AI 生图。
- 动画：视觉与配乐共用时间表，每帧采样 8 个相邻时刻并混合为运动模糊。
- 音乐 / 配音：SoundFont 音色库按代码谱曲，无 TTS 旁白。
- 输出：Skia 像素帧直接送 FFmpeg 合成 MP4，不需要浏览器或部署网站。
- 证据：2026-10-22 已检查上传包实际绘图与导出代码；当前索引未保存该上传包的可下载地址。

## 相关工具，不计入成片数量
- painted-animation-agent-kit：已确认仓库 https://github.com/SherlockGy/painted-animation-agent-kit 。属于 p5.js / p5.brush 手绘工具家族，不是全部项目的共同基础库；接收现成音频，没有完整作曲与配音生成模块。2026-10-21 已提交文字跟随变换与交付前校验工具，语法 / 模拟检查通过，真实像素对比与开销验收当时未完成。
- video-slide-fixer：已讨论的现有视频后处理工具，不是生成画面的引擎；本次检索未恢复精确仓库 URL，暂不填写。

## 横向使用原则
- FFmpeg 是编码 / 合音轨环节，不是“统一绘图引擎”。
- 同源项目通常各保留代码副本，修改一个仓库不会自动同步。
- 声音总体记录显示，多数 Python 动画和 Agent 片用 SoundFont；你好，小生命与南京片更多用 NumPy 合成；配音片大量使用 MiniMax。但这份总体描述不替代每部片的音源核对。
- 讨论统一方案时，可先比较时间轴、音频接口、导出和检查流程；这只是技术建议，未形成全面迁移决定。
- 新增项目或版本时补充：准确仓库 / 文件、成片版本、引擎、素材、驱动、声音、导出、证据日期及已验证范围。
