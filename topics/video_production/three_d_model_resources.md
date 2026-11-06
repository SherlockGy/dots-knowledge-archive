# Hugging Face 3D 模型资源：ABO 与 SimReady

版本：v1.0。建立、最后更新及公开资料核查日期：2026-11-06。

范围：归档资源入口、来源关系、格式、容量口径、用途与许可；不镜像模型，不代表已经下载、导入或实测全部资产。数量和体积是核查时的页面／作者报告值，之后使用应复核。

## 快速选用

| 资源 | 定位与内容 | 核查时规模 | 格式与入口 |
| --- | --- | --- | --- |
| [codepawl/abo-glb][abo] | Amazon Berkeley Objects（ABO）的社区优化版，以家具、家居和商品物体为主 | 7,953 个；基础 WebP GLB 约 12.2 GB，HF 整库约 43.1 GB | GLB；[文件目录][abo-files] |
| [codepawl/simready-glb][sim-glb] | NVIDIA SimReady 的社区转换版，含工业、工具、日用品等物体 | 2,595 个；基础 GLB 约 3.2 GB，含其他变体的整库约 10.9 GB | GLB；[文件目录][sim-files]、[3D 浏览器][sim-viewer] |
| [nvidia/simready-assets][nvidia] | NVIDIA 官方仿真资产源库，也是上一项的上游 | HF 展示 2,677 项、整库约 490 GB | USD；[文件目录][nvidia-files] |

**选用建议（根据格式与资料作出的判断，非本次性能测试）：** 家居布景先查 ABO；网页／Three.js 快速搭景先查两个 GLB 库；需要原始 USD 仿真工作流，优先查 NVIDIA 官方库。两个 SimReady 入口有来源重叠，不能将数量相加当作互不重复的资产。

## 1. ABO GLB：商品与家居模型

原始数据来自 [Amazon Berkeley Objects][abo-original]，CodePawl 做转换优化，不是 Amazon 官方发布的 GLB 压缩包。主要目录：

- `objects/<id>/<id>.glb`：WebP 贴图、meshopt 几何压缩。
- `objects_ktx2/`、`lods/`：KTX2 贴图与较低细节版本。
- `engine/`、`collision/`：未压缩几何的引擎版本与分块碰撞形状。
- `metadata.jsonl`、`thumbs/`：商品资料、模型统计与预览。

数据卡注明米制、Y-up；部分模型并非落在 Y=0。模型没有骨架和独立活动部件，不能直接开门／抽拉抽屉。需要原始高分辨率纹理时回到 ABO 上游。[数据卡][abo]

**容量提醒：** 数据卡下载段仍写“全部约 12.2 GB”，但这对应基础 `objects/`；文件页整库已约 43.1 GB。实际下载应明确选取目录，不能据基础包数字估算全库。[文件目录][abo-files]

## 2. SimReady GLB：适合网页加载的转换版

CodePawl 从 NVIDIA SimReady USD 转为 GLB。主入口为 `glb/<asset>.glb`，另有 `glb_ktx2/`、`lods/`、`manifest.jsonl` 及 `newton_loader/`。[数据卡][sim-glb]

- Three.js 加载 meshopt 版本需要 `MeshoptDecoder`；KTX2 版本另需相应纹理解码支持。
- 当前数据卡报告多数文件带刚体数据，部分带关节；不能笼统理解为“只有静态外观”。
- 同样不能认为 GLB 与原始 USD 完全等价：USD 仿真设置、MDL 专有效果不会完整保留；物理扩展仍有兼容性边界，查看器能够显示不等于物理引擎能够正确模拟。
- 本次没有验证导入、物理稳定性或手机性能；按目标软件与单个资产试用。[数据卡][sim-glb]

## 3. NVIDIA SimReady：官方来源与原始资产

官方库用于查找 USD 原始资源与逐资产元数据。抽查的资产目录含 `simready_usd/`、`.metadata/` 及包装清单；其 [.metadata/LICENSE.txt][nvidia-license] 为 CC BY 4.0。实际使用仍应检查选中资产的许可和依赖文件。

官方展示 2,677 项，转换版展示 2,595 个；本次未逐项比较覆盖范围或版本差异，不推断差额原因。[官方入口][nvidia]

## 许可与署名

三个 HF 页面均标注 **CC BY 4.0**；ABO 官方页面与 NVIDIA 抽查资产的许可文本也已核对。它允许商业使用、改编和再分发，但不是无条件使用或 CC0：

1. 保留提供方指定的作者、署名对象、版权／许可及免责声明，提供原资源与许可链接。
2. 标明自己的改动，并保留已有改动说明；使用 CodePawl 版时同时记录原作者与转换来源。
3. 不暗示 Amazon、NVIDIA 或其他权利人背书，不用额外法律／技术限制妨碍许可授予的权利。
4. 商标、专利及其他独立权利不因此自动获得授权；具体资产或用途有额外问题时单独核查。

ABO 原始数据署名 Amazon.com；数据集构建者的完整署名要求见[官方 Attribution][abo-original]，论文使用还应按该页引用 ABO 论文。SimReady 保留 NVIDIA 原作者信息及所选资产提供的通知。[CC BY 4.0 摘要][cc-by]、[法律文本][nvidia-license]

## 后续使用与维护

- 先按物体名／类别找候选，只取需要的资产和依赖，不默认下载整库。
- 记录资源 ID、所用文件、HF revision、原作者、转换来源、许可及自己的修改，随工程保留。
- 在目标引擎检查比例、朝向、材质、部件、碰撞与解码支持，再决定是否用于成片或仿真。
- 相关成片管线参见[视频制作方案与项目](video_project_tech_index.md)；本条目是素材来源，不是成片引擎或完整制作方案。

## 来源与核查边界

以下固定提交为核查时各 HF 文件页所示 revision，可用于回查；可变入口见上文。本次读取数据卡、文件目录、ABO 官方说明、CC 许可摘要并抽查 NVIDIA 单项许可，没有逐一核验模型文件、全量逐项许可或作者性能测试。

- ABO GLB：[c7c8293fed3b7ab122ef5d7896521a187e0962d7][abo-revision]
- SimReady GLB：[fe284e105db3a58c18f590bf2d7425c968a7bbb1][sim-revision]
- NVIDIA SimReady：[7879e0e50423290300ac20ecd6e727325f680b60][nvidia-revision]

## 版本记录

- 2026-11-06 · v1.0：归档三个资源；区分官方源库与社区转换版、基础包与整库容量；补充许可、使用边界和原始来源。仅保存参考资料与链接。

## 说人话

做场景先挑需要的 GLB，做原始仿真查 NVIDIA USD；SimReady 两个链接是上下游。下载前看清容量，使用时保留署名和许可。

[abo]: https://huggingface.co/datasets/codepawl/abo-glb
[abo-files]: https://huggingface.co/datasets/codepawl/abo-glb/tree/main
[abo-original]: https://amazon-berkeley-objects.s3.amazonaws.com/index.html
[abo-revision]: https://huggingface.co/datasets/codepawl/abo-glb/commit/c7c8293fed3b7ab122ef5d7896521a187e0962d7
[sim-glb]: https://huggingface.co/datasets/codepawl/simready-glb
[sim-files]: https://huggingface.co/datasets/codepawl/simready-glb/tree/main
[sim-viewer]: https://huggingface.co/spaces/codepawl/simready-glb-viewer
[sim-revision]: https://huggingface.co/datasets/codepawl/simready-glb/commit/fe284e105db3a58c18f590bf2d7425c968a7bbb1
[nvidia]: https://huggingface.co/datasets/nvidia/simready-assets
[nvidia-files]: https://huggingface.co/datasets/nvidia/simready-assets/tree/main
[nvidia-license]: https://huggingface.co/datasets/nvidia/simready-assets/blob/main/CT_injection_system_a01/.metadata/LICENSE.txt
[nvidia-revision]: https://huggingface.co/datasets/nvidia/simready-assets/commit/7879e0e50423290300ac20ecd6e727325f680b60
[cc-by]: https://creativecommons.org/licenses/by/4.0/
