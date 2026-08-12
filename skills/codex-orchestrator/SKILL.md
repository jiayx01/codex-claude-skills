---
name: codex-orchestrator
description: 把执行任务委派给 Codex（Claude 规划与验收）。仅当用户点名 Codex 时加载；任务再大，用户没提 Codex 就自己干。
---

# Codex Orchestrator：Claude 规划，Codex 执行

你做规划、拆解、验收，执行默认委派给 Codex。任务书怎么写、`--write`/`--background`/`--resume` 怎么选，官方插件自带的 `codex:codex-cli-runtime` 和 `codex:gpt-5-4-prompting` 已经管了，别重复操心。以下只是那两个 skill 覆盖不到的东西。

用不用 Codex 由用户自己判断，别替他算经济账、别劝阻。

## 调用路径

- **目标项目 == 会话 cwd**：`Agent` 工具，`subagent_type: "codex:codex-rescue"`
- **目标在 cwd 外**：Bash 直调 companion，带 `--cwd`（`--cwd` 不在 rescue 的 flag 剥离白名单里，走 Agent 路径写它会混进 prompt 文本）

```bash
COMPANION=$(ls ~/.claude/plugins/cache/openai-codex/codex/*/scripts/codex-companion.mjs 2>/dev/null | sort -V | tail -1)
node "$COMPANION" task "<任务书>" --write --model <你的模型> --effort <你的档位> --cwd <目标项目根>
```

`--model`/`--effort` 换成你自己账号或网关实际支持的值。

**绝不用 `Skill(codex:rescue)`**——会重入 slash command 挂死会话。`/codex:rescue` 是用户手打的入口。

**effort 档位**：companion 的 `--effort` 白名单目前只到 `xhigh`，传 `ultra` 会直接抛错；要用 ultra 只能不传这个参数，走 config 里的默认值。

## 沙箱：可写根 = cwd

companion 硬编码 `sandbox: write ? "workspace-write" : "read-only"`，**覆盖** `~/.codex/config.toml` 里的 `danger-full-access`。所以可写根是传入的 cwd，**不是任务书里写的路径**——目标在 cwd 外时要么写失败，要么写错位置还自报完成（实测发生过）。

## 后台取结果

```bash
node "$COMPANION" status <job-id>     # 进度，或 --wait --timeout-ms <ms> 阻塞等
node "$COMPANION" result <job-id>     # 最终产出
```

job 按 workspace root 存：委派时带了 `--cwd`，取结果也必须在同一 cwd 下跑。`/codex:status`、`/codex:result` 是 `disable-model-invocation`，你只能 Bash 直调。

互不依赖的任务可以并行派多个 Agent，但**同一仓库并行写会冲突**——同仓库串行，或先拆到不同目录/分支。

## 验收（rescue 明确不做，是你的活）

1. **自己重跑**测试/复现脚本，不采信 Codex 自报的「已通过」
2. **读 diff**，要 Codex 给出改动的 file:line 清单，核对：范围没超任务书（没动无关文件、没删测试蒙混）、改动确实落在目标目录
3. 实现方案与你的规划冲突时，判断优劣再接受

不过关看性质决定下一轮：**局部没改对** → 带反馈 `--resume`；**方向错了** → `--fresh` 重开，resume 会带着错误上下文越修越偏。

**委派失败无产出也计入轮次**（网关 5xx、沙箱写不进这类基础设施故障连续两次就停），两轮不行自己接手收尾，故障情况如实告诉用户。

给 Codex 的任务书里补一条 `gpt-5-4-prompting` 不会替你想的：**它拿不到你的对话上下文**——涉及文件给绝对路径、写清你已经试过什么和为什么失败，否则它会重犯。
