---
name: screenshot-critique
description: 在接受视觉工作之前，把未受暗示的子智能体当作第二双眼睛——在宣布任何用户报告的视觉 bug 已修复或声称视觉变更已验证之前必须执行；受过暗示的眼睛会放过新眼睛能抓到的缺陷。
---

# 截图点评

在接受视觉工作之前，用一个未受暗示的子智能体当第二双眼睛。它针对视觉缺陷，不是像素度量；还需要数字时与 `compare-screenshots` 配对——包括在一张没有对照物的孤立截图上，后者的场景度量能说出画面里到底有没有内容。

## 工作流

1. 截取或定位受审的确切 PNG/GIF 帧。
2. 保留原生尺寸证据，并为每个关键特征制作 2x-4x 的补充裁剪，外加完整截图。包含完整的端点和淡出；几何变化后复查裁剪边界。裁剪选中的单位、城市/城镇群、旗帜/旗杆、阴影、选中环、标签/图标、道路、地形特征、水体，以及任何易出伪影的区域。如果投诉是关于"太淡""顺序不对"或"透视不对"，这个裁剪是强制的。
3. 用 `fork_context: false` 派生一个全新的 explorer；只传完整图像、裁剪和一段简短中立的任务。评判保真度时附上已批准的参考和用户要求；扣留历史、实现细节、既有结论和预期答案。
4. 要求带置信度的具体可见缺陷。点名可能的风险类别：单位/道具深度排序、分层、阴影、选中标记对比度、地面透视、旗帜/旗杆附着、标签样式与图标可读性、模糊、尺度、光照、伪影、缺失模型、地形特征可读性、道路、水体和整体画面可读性。
5. 把子智能体的点评与你自己的检查对比。把重叠当作高优先级证据。把新颖的高置信度发现当作待检查的 bug，而不是可以打发掉的品味意见。
6. 在宣布截图被接受之前，把可执行的发现记录进规格说明、视觉报告或下一个任务计划。

## 子智能体提示词

使用这个形状，替换方括号里的界面并附加本地图像：

```text
Fresh visual critique task. You have no project backstory and should only
inspect the supplied screenshots and crops. First inspect the full screenshot
for context, then inspect each crop at zoomed scale. Look for concrete
visual/layout defects in [surface], especially unit/prop depth ordering,
layering, shadows, selection-marker contrast, ground-plane perspective,
flag/pole attachment, label style/icons, blur, scale, lighting, artifacts,
missing models, terrain feature readability, roads, water, and scan
readability. Do not assume these are correct. Return a concise list of issues
you can see, with confidence and whether the issue is visible in the full image,
the crop, or both.
```

派生配置：

- `agent_type`: `explorer`
- `fork_context`: `false`
- 把截图作为 `local_image` 项附加
- 除非用户明确要求，否则省略模型覆盖

## 规则

- **"已修复"之前强制执行：** 绝不凭自己的检查宣布用户报告的视觉 bug 已修复——你的眼睛已被你刚做的修复暗示过。先对候选截图运行未受暗示的点评；你打算放行的"轻微残留"正是它存在要抓的东西。（记录在案的失败：一片"已修复"的天空，被一个未受暗示的智能体认出是地形网格的底面填满了整个天空区域。）
- **复现报告者的取景。** 用户提供了截图时，点评必须包含一张该取景的截图（相同相机/缩放/位置，或尽可能接近可复现）——活在他们的取景里的缺陷，在你的取景里可能不可见。你自选的探查取景是补充，绝不是替代。
- **先证明变更是真的，再点评它。** 先把候选与改前基线做字节/像素 diff：对一张未变的图像做点评，等于"验证"了一次空操作。（记录在案的失败：一次从未到达生产渲染路径的调色——改前改后字节完全相同，只有 diff 抓住了它。）
- **交接完整的截图集合，绝不是精选过的。** 你截取的每个状态、每个 viewport、桌面与移动端。挑选展示哪些截图，正是这次全新检查要消除的那种偏见：你会挑你本来就觉得没问题的，而弱的状态恰好就是被落下的那个。某个状态手工难以到达时，确定性地驱动它并截取，而不是省略。
- **没有子智能体可用时，自己替对方辩护。** 对每个受审特征，写一句话，只用截图中可见的东西构建"它是坏的"的最强论证——然后再决定。先写论证才使检查有对抗性；先决定再找理由，正是这个技能要替代的受暗示检查。把这些句子放进报告，让推理可评审。
- 绝不告诉子智能体你期望它找到什么缺陷。
- 使用当前候选截图，不是过时的报告或基线图。
- 不要依赖整页报告的尺度看小型视觉特征。在玩家会读到的确切特征周围附加裁剪：选中的军队/城市、标签/图标簇、旗帜、阴影、环边缘、道路交叉口、地形特征块、水体标签，以及可疑的调试/伪影区域。
- 如果子智能体说某个裁剪暴露了一个在全图里微弱或不可见的问题，当玩家可以在游戏内缩放到该尺度时，把它当作真实的可用性缺陷。
- 动画先附一小串确定性的静帧；GIF 对人类评审有用，但静帧让具体缺陷更容易被点名。
- 一个通过的标题不会抹掉一个被报告的小失配或不确定。用[参考基准点](../compare-screenshots/references/reference-landmarks.md)解决它；第二意见不替代直接检查或回归门禁。
- 如果子智能体抓到了主智能体漏掉的问题，立刻把那个失败模式加进相关特征计划或视觉清单。
