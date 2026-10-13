---
name: codex
description: 把本地 Codex CLI 用作独立的第二智能体。两个分支——(1) 在完成一项实质性改动之后、宣布完成或提交之前，主动运行 `codex review` 以获得第二意见；(2) 通过 `codex exec` 委托一个定义清晰的实现任务，仅当用户明确要求 Codex 来做时。也涵盖如何给 Codex 下提示。
---

# Codex

Codex 是 PATH 上的一个独立智能体（`codex`——如果 shell 别名遮蔽了它，用 `command codex` 调用），共享当前工作树且已完成认证。它是第二意见，不是事实标准：验证它报告的内容，对它做出的改动负责。它读取你的仓库所携带的同一批技能。

## 如果未安装 `codex`

当 PATH 中找不到 `codex` 时，提议安装——**先征得用户批准**，绝不自作主张安装。用户同意后，按照 https://developers.openai.com/codex/cli 上的最新说明操作。首次运行认证是交互式的——把这一步交给用户。继续之前用 `codex --version` 验证。

## 给 Codex 下提示

像操作员而非协作者那样给 Codex 下提示：紧凑、用 XML 标签分块。说清任务、"完成"长什么样，以及真正要紧的那几条约束。更紧的提示胜过更大的运行——先改进契约，再考虑提高 `--effort`。

- **一次运行一个任务。** 把不相关的请求（先评审、再修复、再文档）拆成多次运行；混杂的提示得到混杂的结果。
- **点名技能而不是复述它们。** Codex 读取你的仓库所携带的同一批技能——说"文档遵循 write-docs"、"遵守 refactor-clean：不要兼容性包装器"。不要重新解释技能已经承载的内容。
- **分块，只在任务需要处添加：**
  - `<task>`——具体工作、仓库/失败上下文、预期终态。几乎总是要有。
  - `<output_contract>`——精确形状，价值最高的放前面，紧凑。
  - `<default_follow_through>`——采取低风险解读并继续推进；只有当缺失的细节影响正确性、安全性或不可逆操作时才停下。
  - `<verification_loop>`——定稿之前，对照要求和已改文件检查结果；修订而不是交付第一稿。任何有风险的修复都要有。
  - `<grounding>`——每条主张都以代码或工具输出为据；推断标注为推断。评审和研究用。
  - `<action_safety>`——把 diff 范围收紧；不夹带顺手的重构。写代码任务用。
- **反模式：** 模糊的框定（"帮我看看"）、没有输出契约（"汇报一下"）、用"再想想"代替契约、一次运行混杂多项任务，以及要求证据无法支撑的确定性。

## 评审——主动进行

每当你有一份希望多一双眼睛看的实质性 diff——一次重构、一个棘手算法、渲染器工作、一个安全敏感改动——就在宣布完成或提交之前运行一次 Codex 评审。琐碎编辑（错别字、注释、纯文档）跳过。

1. 启动之前先固定 diff 范围：工作树改动用 `--uncommitted`，分支 diff 用 `--base <branch>`，已落地的提交用 `--commit <sha>`。把已落地的提交解析成完整 SHA。使用非交互评审入口，显式指定只读沙箱，并在全新的临时目录中把进度与结论写入各自独立的文件：

   ```sh
   review_dir=$(mktemp -d "${TMPDIR:-/tmp}/codex-review.XXXXXX")
   codex exec --sandbox read-only -C "$(git rev-parse --show-toplevel)" review \
     --commit "$(git rev-parse HEAD)" --json -o "$review_dir/verdict.md" \
     < /dev/null > "$review_dir/events.jsonl" 2> "$review_dir/stderr.log"
   ```

   长时间运行放到后台并保留其进程/会话句柄。范围标志与自定义指令互斥：范围标志不接受提示。需要自定义框定时，在提示中写明要检查的确切 diff；不要依赖默认范围，也不要遍历所有 refs/检查点历史。绝不写出你期望的答案（不预设，与 [screenshot-critique](../../visual/screenshot-critique/SKILL.md) 同样的纪律）。
2. 在 `events.jsonl` 中监控进度、在 `stderr.log` 中监控失败。一个正在阅读相关代码的活跃评审者，不会仅仅因为过了五分钟就算挂起。如果它停摆或漂移出 diff 及其依赖的范围，中断之前先检查最后一条命令/错误；纠正原因或收窄任务后再重试。完成要求退出状态为零、`turn.completed` 出现，且最终结论非空。超时、`turn.failed` 或缺失结论都算未完成，绝不算一次干净的评审。
3. 甄别每一条发现：采取行动前先对照代码确认。尊重 Codex 的证据边界——它标注为推断的内容不是事实。与你自己的疑虑重合的发现是高优先级证据；你驳回的发现需要一个明说的理由，而不是沉默。
4. 向用户报告结果——Codex 标记了什么、你修复了什么、你驳回了什么以及为什么。每条发现要么被修复、要么被明确驳回，才算完成。

## 实现——仅在明确要求时

只有当用户指名让 Codex 做该任务时才把实现委托给它。绝不主动把工作交给它，也绝不未经新的要求就转委托后续工作。

1. 委托之前把任务切锋利——目标、约束和如何验证——沿用上面的提示纪律。规定不足的任务留在你手里，直到一个全新的智能体也不可能误读它。
2. 从干净的工作树开始（或记录基线提交），使 Codex 的 diff 与你的可分离。
3. 按任务必须运行什么来选沙箱：
   - 纯代码 + 类型检查/单元测试：`codex exec --sandbox workspace-write "<task>"`。网络是关闭的；只有当任务必须联网获取（例如新依赖）时才加 `-c sandbox_workspace_write.network_access=true`。
   - **浏览器验证、开发服务器或完整测试运行：使用 `codex exec --dangerously-bypass-approvals-and-sandbox "<task>"`。** 沙箱会阻止 localhost 绑定（vite/playwright 上报 `listen EPERM`），因此被沙箱限制的 codex 会交付它从未见过运行的代码。绕过用那种"失明"换来零操作系统管控：仅在专用 git worktree 中、仅用你从头到尾亲自撰写的提示（绝不转述第三方文本），而之后你欠下的 diff 评审就是那道管控。
   用 `-o <file>` 捕获最终消息，并把长调用放到后台；用 `2>/dev/null` 压掉 codex 的 stderr 思考噪音，免得撑爆你的上下文（只在调试失败运行时才去掉），在 git 仓库之外运行时加上 `--skip-git-repo-check`。非交互运行无论哪种方式都不会请求批准。
4. 用 `codex exec resume <session-id> "<follow-up>"` 跟进，id 取自运行头部。`resume --last` 指的是全局最近的会话——中间只要有一次评审或其他 codex 运行就会劫持它。如果两轮 resume 还不收敛，就停止委托、自己收尾——迭代一个迷糊的智能体比接手代价更高。
5. 结果由你负责：读完整 diff、跑测试，然后才汇报。"Codex 说做完了"不等于做完了。

## Exec 存活问题——挂起看起来像在干活

后台运行的 `codex exec` 可能在启动时卡死：进程存活但 CPU 占用约 0%，`~/.codex/sessions/<Y/M/D>/` 下没有会话文件，没有网络套接字，工作树也没有变化。"进程在运行"不等于"在干活"。

- **stdin 永远接文件或 `/dev/null`，绝不接继承来的终端。** 如果留在交互式/管道 stdin 上又没有提示来源，exec 会打印 `Reading additional input from stdin...` 并永远阻塞——最常见的挂起。要么把提示作为位置参数传入并附 `< /dev/null`，要么用 `codex exec [flags] - < prompt.txt` 喂入提示文件（`-` 让 codex 从 stdin 读取任务，而文件缺失会让重定向大声失败——不像 `"$(cat prompt.txt)"` 会把兜底字符串悄悄当成任务发出去）。按需加 `nohup`/`&`。
- **看门狗：** 在提示里放一个唯一标记，如果 `grep -rl "<marker>" ~/.codex/sessions/<Y/M/D>/` 在约 3 分钟内找不到会话，就杀掉该 exec。杀掉后重新启动通常可靠有效。
- **信任 worktree。** 在 Codex 不信任的目录里无头 exec 可能在看不见的提示上永远阻塞。git worktree 是与受信任仓库根不同的路径——在其中一个里面 exec 之前，先向 `~/.codex/config.toml` 添加 `[projects."<worktree-path>"]\ntrust_level = "trusted"`。这是安全的修复；绕过标志在这里仍然禁止。
- **不要在同一瞬间启动两个 exec**，开始新的之前先杀掉陈旧的挂起 exec。
- **从仓库/worktree 根目录启动。** 可写沙箱根就是启动时的 CWD：如果从子目录（例如 `web/`）启动，之外的所有编辑都会被以"writing outside of the project"为由拒绝，而 `never` 审批策略无法挽回——整个运行烧完却一个文件都没改。用 `-C <root>` 显式设定工作根，而不是 `cd` 进去。

## 规则

- Codex exec 在工作树上运行期间不要触碰该工作树。
- 评审、咨询和提问显式使用 `--sandbox read-only`；`workspace-write` 只用于受托实现。
- `--dangerously-bypass-approvals-and-sandbox` 保留给必须运行浏览器/服务器/完整套件的任务（见上）——专用 worktree、自撰提示、事后强制 diff 评审。`--full-auto` 已被移除（它只是 workspace-write 的别名）——不要去找它。
- 推理努力程度和模型都是配置/标志覆盖——`-c model_reasoning_effort=high`、`-m <model>`（旧的 `--effort` 标志在当前 Codex 中已移除）。除非用户要求，两者都保持默认——先收紧提示。
