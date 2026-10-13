# 发布视频

一部约 35 秒 showreel 风格的本仓库发布视频，用 [launch-video](../skills/graphics/launch-video/SKILL.md) 技能制作。概念生长自主视觉的*战争迷雾*：一片被迷雾笼罩的六边形大陆先被测绘（`/explore-unknowns`），再切分为领地（`/write-spec`），并在夜里于其上施工，同时一个智能体验证每片领地（`/goal /implement-spec`）。片尾落在 "the factory runs itself" 上，然后收束为字标（wordmark）的句点。

```bash
npm install
npm run audio    # synthesize + master public/soundtrack.wav (-12 LUFS, -1 dBTP)
npm run studio   # scrub it in the browser
npm run draft    # half-scale review render
npm run render   # 1080p60 H.264, 320k AAC → out/skills-launch.mp4
npm run still -- KeyArt out/key-art.png   # the headline-number still
```

这个世界用 three.js 构建（经 `@remotion/three`），所以渲染需要 GPU 支持的浏览器（`--gl=angle`，已写入脚本）。渲染产物和生成的音频被忽略；新 clone 之后重跑 `npm run audio`。

## 同一个计时源

`src/cues.ts` 以 120 BPM 的歌曲秒数保存每一个事件时间。3D 世界、文字层和 `audio/synth.ts` 都读它，所以移动一个 cue，它的画面与声音一起移动。先以 cue 的形式加入视觉事件，再为它配上声音。一切都是歌曲时间的纯函数；没有任何东西读取帧。

前 0.3 秒是一段预卷，把完成的安装画面定格为信息流缩略图；`PREROLL` 同时平移画面与音乐。

## 必须保持的论断

- "The factory runs itself"（工厂自己运转）转述根目录 README 中完整循环的无人值守中段（"software factories"）。保持它与原文的绑定。
- 无人值守计时器从小时一路数进天数，是对一次长时自主运行的示意；它不是实测论断。
- "Claude Code · Codex · +70 harnesses"（+70 个宿主环境）来自根目录 README。

音频靠测量验证（ffmpeg `ebur128` 与频谱图），不是靠耳朵。
