# Claude-Mem 代码库学习路线

> 组织原则：**从整体到细节，从使用到原理**。
> 先当用户把它跑起来看现象 → 再当集成者理解它怎么接入 → 最后当开发者拆到实现原理。
> 每一层都建立在上一层的直觉之上。

## 学习节奏总览

```
第1-2层：使用 + 整体       ← 黑盒/灰盒，不碰实现，建立直觉
第3层：  接入接口          ← 使用与原理的交界
第4-7层：原理，从数据到编排 ← 细节，沿主线逐环深入
第8层：  分支专题          ← 按兴趣
```

一句话概括：**先看它做了什么（1-2）→ 再看它从哪被调起（3）→ 然后固定数据形状（4）→ 沿主线拆写入（5）和读取（6）→ 最后理解谁在编排（7）。**

---

## 贯穿全程的主线问题

> 「我在会话里编辑了一个文件 → 这条操作如何变成下次会话开头的一行上下文？」

它依次经过：

```
hooks.json（触发）
  → observation 处理器（捕获）
    → transcripts/（读会话记录）
      → sdk/（压缩）
        → sqlite/ + chroma（落库）
          → 下次会话 context（检索）
            → ContextBuilder（组装注入）
```

建议：把这句话写在纸上，每深入一个模块就在这条链上标记「现在读到哪一环」。

---

## 第一层 · 会用它（黑盒视角：只看现象，不看代码）

> 目标：先有「它到底帮我做了什么」的肌肉记忆，后面所有代码都有落点。
> 📖 **本层完整讲义：[`01-会用它-黑盒视角.md`](./01-会用它-黑盒视角.md)**

- [x] 1. 读产品介绍，建立一句话认知：跨会话记忆 —— `README.md`
- [x] 2. 亲自装一次、跑一次，观察真实产物 —— `~/.claude-mem/claude-mem.db`（`observations` 表）、`settings.json`、`worker.pid`
- [x] 3. 用自带技能体验完整闭环 —— `plugin/skills/how-it-works`、`mem-search`、`timeline-report`
- [x] 4. 打开后台 UI 看记忆/时间线 —— worker 的 `GET /` 与 `/stream`（见 `src/services/worker/README.md`）

**结束标准**：能回答「一次会话结束后，我的哪些操作被记住了？下次会话开头多出来的那段上下文长什么样？」

---

## 第二层 · 懂它的整体结构（灰盒视角：一张架构图 + 一条主数据流）

> 目标：不看具体函数，先在脑中建立「进程模型 + 数据流向」。
> 📖 **本层完整讲义：[`02-整体结构-灰盒视角.md`](./02-整体结构-灰盒视角.md)**

- [x] 5. ⭐ 读懂运行时全景：常驻 worker、端口、请求流、路由分层 —— `src/services/worker/README.md`
- [x] 6. 读懂目录约定：源码→构建→安装的三段式 —— `CLAUDE.md` + `package.json` 的 `scripts`
- [x] 7. 抓住那条主数据流（写入→读取闭环）—— 见上方【主线问题】

---

## 第三层 · 懂它怎么接入 Claude Code（接口视角：使用与原理的交界）

> 目标：理解「插件是如何被 Claude Code 调起来的」——这是使用与原理的分水岭。
> 📖 **本层完整讲义：[`03-接入接口.md`](./03-接入接口.md)**

- [x] 8. ⭐ hook 生命周期总入口：谁在什么时机被调 —— `plugin/hooks/hooks.json`（看 `SessionStart / UserPromptSubmit / PostToolUse`）
- [x] 9. hook 命令分发与统一响应 —— `src/cli/hook-command.ts`、`src/hooks/hook-response.ts`
- [x] 10. 每个 hook 对应的处理器（按触发顺序读）—— `src/cli/handlers/`：`session-init → context → user-message → observation → file-edit → summarize`

**结束标准**：能回答「Claude Code 每发生一件事，claude-mem 的哪段代码被执行？入参出参是什么？」

---

## 第四层 · 懂数据模型（原理的地基：先看「记忆长什么样」）

> 目标：在读读写逻辑之前，先固定住数据结构，否则后面会飘。
> 📖 **本层完整讲义：[`04-数据模型.md`](./04-数据模型.md)**

- [x] 11. ⭐ 记忆的核心数据结构 —— `src/core/schemas/`：`memory-item.ts`、`session.ts`、`project.ts`、`agent-event.ts`
- [x] 12. 落库表结构与读写 —— `src/storage/sqlite/`：`schema.ts` → `memory-items.ts` → `projects.ts` → `serde.ts`
- [x] 13. 对照第二套后端理解存储抽象 —— `src/storage/postgres/`（cloud-sync 用）

---

## 第五层 · 懂写入原理（记忆是怎么「生成」的）

> 目标：主线的前半段——从原始会话记录到结构化记忆。

- [ ] 14. ⭐ 会话记录监听与处理 —— `src/services/transcripts/`：`watcher.ts → processor.ts → state.ts`
- [ ] 15. ⭐ 压缩核心：提示词 + 输出解析 —— `src/sdk/`：`prompts.ts`、`parser.ts`、`output-classifier.ts`、`hardened-options.ts`
- [ ] 16. 多模型 Provider 适配与容错 —— `src/services/worker/`：`ClaudeProvider.ts`、`GeminiProvider.ts`、`provider-errors.ts`、`retry.ts`

---

## 第六层 · 懂读取原理（记忆是怎么「找回并注入」的）

> 目标：主线的后半段——从检索到上下文注入。

- [ ] 17. 检索逻辑（含向量检索）—— `src/services/worker/SearchManager.ts` + `search/`
- [ ] 18. ⭐ 上下文组装与 token 预算 —— `src/services/context/`：`ContextBuilder.ts → ObservationCompiler.ts → TokenCalculator.ts → sections/`
- [ ] 19. 以 MCP 工具形式对外暴露检索 —— `src/servers/mcp-server.ts`、`src/server/mcp/recall-mcp-server.ts`

---

## 第七层 · 懂运行时编排（把所有能力串起来的后台进程）

> 目标：理解那个常驻 HTTP 服务如何调度前面所有模块。

- [ ] 20. 路由层（对照第 5 步的 README）—— `src/services/worker/http/routes/`
- [ ] 21. 会话状态与实时推送 —— `SessionManager.ts`、`SessionMessageBuffer.ts`、`SSEBroadcaster.ts`
- [ ] 22. worker 启停与进程守护 —— `worker-service.ts`、`worker-spawner.ts`、`src/supervisor/`

---

## 第八层 · 进阶专题（按需选读，不影响主线）

- [ ] 云端同步（近期主线）—— `src/services/sync/`、`plugin/skills/cloud-sync/`
- [ ] 知识 corpus —— `src/services/worker/knowledge/`、`plugin/skills/knowledge-agent/`
- [ ] 安装/运维 CLI —— `src/npx-cli/commands/`、`src/services/install/`
- [ ] 跨编辑器适配 —— `src/integrations/opencode-plugin`、`cursor-hooks/`
- [ ] 用测试反向验证理解 —— `tests/` 对应模块

---

## 学习建议

- **主线优先**：第 1→2→3→4→5→6→7 层是一条完整的「写入→读取」闭环，先打通主干，第 8 层都是分支。
- **带着数据流读**：始终用上方的主线问题串起来，每读一个模块就对应到链条上的一环。
- **⭐ 是每层的必读锚点**，其余为展开阅读。
- 每完成一步，把对应的 `[ ]` 勾成 `[x]`。
