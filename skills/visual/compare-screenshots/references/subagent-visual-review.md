# 子智能体视觉评审

当历史或既有结论可能使主智能体的视觉判断带有偏见时使用。

## 派生配置

- `agent_type`: `default`
- `fork_context`: `false`
- 把两张截图作为 `local_image` 项附加。
- 已批准设计的对比中，指明 `Reference` 与 `Candidate`，并提供预期设计要求。开放偏好的对比中，使用 `Image A` 与 `Image B`，不宣布目标。
- 提供完整帧和含周围空间的匹配细节裁剪。扣留既有结论、实现细节和作者偏好的答案；不要扣留判断保真度所需的目标。

## 提示词

```text
You are doing an unbiased visual review of two screenshots for the same visual target. You have no prior context.

Compare Image A and Image B. Report:

1. Whether they appear to show the same viewport/state/content.
2. Major visible differences in camera/view, layout, content, missing details,
   labels/text, icons, color, lighting, depth/layering, clipping, artifacts,
   readability, or style.
3. Inspect every changed effect at all four boundaries, including its full fade
   beyond the component. Compare extent relative to visible anchors on each
   side. Check brightness, color, sharpness and continuity of every foreground
   feature it crosses. Report local discrepancies even if everything is legible.
4. Give a per-feature reference observation → candidate observation →
   pass/fix/uncertain record. Distinguish solid/core edges from faint fringe;
   measure endpoints and gaps relative to visible anchors. Verify crops include
   the entire feature. Request paired effect-on/off captures when background
   texture makes extent ambiguous. A small discrepancy still needs resolution;
   do not label it acceptable merely because the whole looks similar.
5. A concise verdict on whether the images preserve the intended visual
   relationship or need another pass.

When an approved reference is identified, judge fidelity to it. Otherwise do not
assume either image is the target. Overall attractiveness does not excuse a
local mismatch. Judge only from visible pixels and supplied design requirements.
```

## 如何使用结果

- 把子智能体的结果当作关于哪张图错得少的独立证据，不是度量或你自己检查的替代——也不是给基线那张图的一票。
- 如果子智能体标记了相机错误、状态失配、内容缺失或可见伪影，先修截图/渲染质量，再评判其余。
- 保留发现，而不只是标题结论。伴随一个未解决局部差异的"通过"不是接受；在宣布特征完成之前，先做完基准点检查。
