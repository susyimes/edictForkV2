# ⚔️ Edict Fork V2

> 从“能演示三省六部多 Agent 看板”升级到“可持续开发、可观测、可审计、可扩展的多 Agent 协作平台”。

这是我基于原始 **Edict / 三省六部** 项目做的 V2 分支设计与重构版本。
V1 更像一个很有想法、很有表现力的 Demo / 可运行原型；**V2 的目标** 是把它推进成一套真正适合长期演进的工程化系统：

- **事件驱动**，而不是主要依赖脚本轮询
- **前后端分离**，而不是单文件看板 + Python stdlib 服务为主
- **可审计可回放**，所有任务流转都有结构化事件记录
- **可观测**，能看到任务、事件、thought、todo、状态流转
- **可扩展**，后续可接入更多 Agent、更多模型、更多外部系统

---

## V2 想解决什么问题？

原版 Edict 的核心创意非常强：

- 用 **三省六部** 来表达多 Agent 分权制衡
- 有很强的角色感、制度感、可视化体验
- 能把复杂任务拆成“分拣 → 规划 → 审议 → 派发 → 执行 → 回奏”

但如果继续往产品化/平台化走，会遇到几个典型问题：

1. **状态与流程耦合较重**：很多逻辑散落在脚本与看板之间
2. **实时性依赖轮询**：不利于后续扩展到真正的事件流和订阅机制
3. **审计能力不够结构化**：历史流转能看，但还不够适合系统级追踪
4. **V1 与未来架构并存时边界不清**：需要明确 legacy 和 next-gen 的关系
5. **难以做统一控制面**：例如暂停、重试、回放、人工审批、事件追踪等能力

所以 V2 的方向不是“重写一个新 UI”，而是：

> **把 Edict 从“有组织感的多 Agent Demo”升级成“真正可运营的多 Agent 系统内核”。**

---

## V2 的核心设计

V2 保留 **三省六部的业务理念**，但把底层实现切换为更标准的现代系统架构：

### 1. 控制面 / API 层
负责统一提供：

- 任务创建与查询
- Agent 信息查询
- 管理操作
- 事件查询
- WebSocket 实时推送

当前目录：

- `edict/backend/app/main.py`
- `edict/backend/app/api/*`

当前技术选型：

- **FastAPI**
- REST API + WebSocket
- 作为 V2 的统一入口

---

### 2. 任务状态机
任务不再只是看板上的一条记录，而是一个有状态、可演进、可审计的实体。

当前已经在 V2 中落地了基础状态管理：

- 创建任务
- 任务状态流转
- 流转合法性校验
- flow log / progress log / todos / scheduler 等结构化字段

相关目录：

- `edict/backend/app/models/task.py`
- `edict/backend/app/services/task_service.py`

V2 目标是把“皇上 → 太子 → 中书 → 门下 → 尚书 → 六部 → 回奏”的流程，抽象成：

- **明确状态**
- **合法迁移规则**
- **可追踪的流转日志**
- **可回放的事件序列**

这样后续不管 UI 怎么变、Agent 怎么增减，任务主线都不会丢。

---

### 3. 事件驱动架构
这是 V2 最关键的变化。

V1 更偏脚本调度与看板同步；V2 希望转成 **Event-Driven Architecture**。

核心思路：

- 任务创建 → 发布 `task.created`
- 规划请求 → 发布 `task.planning.request`
- 审议结果 → 发布 `task.review.result`
- 派发执行 → 发布 `task.dispatch`
- 思考流 / Todo 更新 → 发布 `agent.thought.append` / `agent.todo.update`
- 状态变更 → 发布 `task.status.*`

这样做的好处：

- 前端、调度器、执行器可以**解耦**
- 更容易接入 **Replay / Trace / Monitoring**
- 更适合横向扩展和异步执行
- 后续要接 Redis Streams / NATS / Kafka 都有清晰演进路径

当前相关目录：

- `edict/backend/app/services/event_bus.py`
- `edict/backend/app/workers/*`
- `edict/backend/app/api/websocket.py`

---

### 4. 前端控制台
V2 不再把前端仅仅视为“展示板”，而是视为 **Control Plane UI**。

目标包括：

- 任务总览（Kanban / Table / Timeline）
- 实时事件流
- 任务详情页
- thoughts / todos 可视化
- Agent 状态观察
- 人工介入（暂停 / 恢复 / 封驳 / 准奏）
- 后续接入审批、回放、审计能力

当前目录：

- `edict/frontend/`

当前技术选型：

- **React 18**
- **TypeScript**
- **Vite**
- **Zustand**
- TailwindCSS

这意味着 V2 的前端已经是一个独立可持续开发的工程，而不是单页演示文件。

---

### 5. 审计与可观测
V2 不是只追求“Agent 能跑起来”，而是更关心：

- 谁创建了任务？
- 任务何时进入哪个状态？
- 是哪个 Agent 做了哪一步？
- 是否被门下省打回？为什么？
- 执行链路里产生了哪些 thought / todo / event？
- 出错后能不能复盘？

这部分是 V2 的长期重点。

一句话概括：

> **V1 更像“看多 Agent 怎么协作”，V2 更像“让多 Agent 协作变成可治理系统”。**

---

## 当前仓库结构

这个仓库现在实际上包含两层内容：

### A. Legacy / V1 资产
仍然保留原有的三省六部看板、脚本、演示素材和安装脚本，方便参考与兼容。

主要包括：

- `agents/` — 原版各省部 SOUL 配置
- `dashboard/` — 原版看板与服务
- `scripts/` — 原版同步、采集、模型切换、技能管理等脚本
- `docs/` — V1 的架构、截图、演示与说明
- `tests/` — V1 相关测试

### B. V2 / 新架构实现
这是当前 fork 里最值得关注的部分：

```text
edict/
├─ backend/                # FastAPI + 服务层 + 状态机 + 事件总线
├─ frontend/               # React 18 控制台
├─ migration/              # 数据迁移脚本
├─ scripts/                # V2 辅助脚本
├─ docker-compose.yml      # 本地联调环境
└─ .env.example
```

如果你是第一次看这个仓库，建议优先关注：

- `edict/backend/app/main.py`
- `edict/backend/app/services/task_service.py`
- `edict/backend/app/services/event_bus.py`
- `edict/backend/app/models/`
- `edict/frontend/src/`
- `edict_agent_architecture.md`

---

## V1 和 V2 的关系

不是推倒重来，而是 **两层演进**：

### V1 的价值
V1 已经验证了很多非常重要的东西：

- 三省六部这个组织模型是成立的
- 用户能直观理解“太子 / 中书 / 门下 / 尚书 / 六部”的协作关系
- 看板、奏折、模板、仪式感这些交互很有表达力
- 多 Agent 系统需要“制度化”而不是只靠自由对话

### V2 的目标
V2 不再主要验证概念，而是解决工程问题：

- 如何让任务系统更稳定？
- 如何让状态流转更清晰？
- 如何让事件可以实时订阅？
- 如何让控制台成为真正的控制面？
- 如何让后续功能（审批、回放、适配器、知识库）可以自然接进去？

所以可以把两者理解成：

- **V1 = 产品概念验证 + 表达层成熟**
- **V2 = 系统内核重构 + 平台层打底**

---

## 当前 V2 进度

### 已有基础
- [x] FastAPI 后端骨架
- [x] API 路由分层（tasks / agents / events / admin / websocket）
- [x] 基础任务模型与状态机
- [x] 任务服务层（create / transition / dispatch / progress / todos）
- [x] 前端 React 工程化目录
- [x] Docker / migration 基础文件
- [x] V2 架构草图与设计文档

### 正在完善
- [ ] 统一事件协议细化
- [ ] worker / orchestrator 调度闭环
- [ ] review / dispatch / execute 的全链路状态推进
- [ ] WebSocket 实时订阅模型完善
- [ ] thought / todo / event 的前端展示统一化
- [ ] legacy 数据向 V2 模型迁移

### 下一阶段重点
- [ ] 门下省审议节点正式产品化
- [ ] 任务时间线回放
- [ ] 人工干预（暂停 / 恢复 / 重试 / 封驳）
- [ ] Agent 绩效与健康监控
- [ ] 知识沉淀 / 国史馆能力

---

## V2 的基础模块设计

下面是当前建议的 V2 最小闭环：

### 模块一：Task Core
负责定义任务实体与状态机。

它至少要回答 4 个问题：

1. 任务现在在哪个阶段？
2. 任务是否允许进入下一个阶段？
3. 历史流转记录是什么？
4. 前端该如何稳定展示？

---

### 模块二：Event Bus
负责承载任务与 Agent 之间的异步事件流。

它至少要支持：

- publish
- subscribe
- replay（后续）
- trace_id 贯通

---

### 模块三：Orchestrator
负责把业务制度落成机器流程。

例如：

- 收到 `task.created` 后，派给太子 / 规划层
- 收到 `planning.complete` 后，进入审议
- 审议通过后，再进入派发
- 派发后跟踪各执行节点完成度
- 所有子任务完成后，进入回奏 / closing

---

### 模块四：Realtime Console
这是前端控制面。

需要能看到：

- 当前任务列表
- 单任务详情
- 事件时间线
- todos
- thoughts
- 执行状态
- 人工操作入口

---

### 模块五：Audit / Replay
这是 V2 区别于普通 Agent Demo 的关键能力之一。

未来应该支持：

- 查看任务完整事件流
- 回放某个任务如何被规划、审议、派发和完成
- 对问题任务做事后分析
- 为绩效、调优、风控提供数据基础

---

## 推荐开发顺序

如果要把 V2 做成真正可跑的主线版本，我建议按下面顺序推进：

### Phase 1：跑通最小任务闭环
- 创建任务
- 状态流转
- 事件发布
- WebSocket 推送
- 前端可见

### Phase 2：补齐三省流程
- planner（中书）
- reviewer（门下）
- dispatcher（尚书）
- executor（六部）

### Phase 3：做强控制面
- timeline
- replay
- pause / resume / retry
- manual approval

### Phase 4：兼容 legacy 与生态扩展
- 数据迁移
- 与现有 OpenClaw / scripts / dashboard 兼容
- 更多外部适配器

---

## 为什么我认为 V2 值得做

因为这个项目真正特别的地方，从来不只是“古风命名很有趣”。

它真正稀缺的是：

> **它试图把多 Agent 从“大家一起聊聊看”推进成“有制度、有层级、有制衡、有审计的协作系统”。**

而要让这个方向走得更远，就必须从：

- Demo 思维
- 脚本拼装
- 面向展示

转到：

- 平台思维
- 事件驱动
- 面向治理

这就是 V2 的意义。

---

## 快速开始（针对当前 V2 目录）

> 注意：V2 目前仍处于基础搭建阶段，下面更适合开发者本地阅读与联调，不等同于“生产可用安装说明”。

### 1. 克隆仓库

```bash
git clone https://github.com/susyimes/edictForkV2.git
cd edictForkV2
```

### 2. 查看 V2 目录

```bash
cd edict
```

建议先阅读：

- `backend/app/main.py`
- `backend/app/services/task_service.py`
- `backend/app/services/event_bus.py`
- `frontend/src/App.tsx`
- `../edict_agent_architecture.md`

### 3. 启动前端 / 后端（按各自目录说明补全）

当前仓库已具备：

- `edict/backend/requirements.txt`
- `edict/frontend/package.json`
- `edict/docker-compose.yml`

后续我会继续补齐更正式的开发启动说明。

---

## 文档

- `edict_agent_architecture.md` — V2 架构草图与设计思路
- `docs/task-dispatch-architecture.md` — V1 / 原版任务流转说明
- `ROADMAP.md` — 路线图（后续会逐步切换为更偏 V2 的表达）

---

## 本仓库当前定位

当前这个 fork 的定位不是“已经完成的产品”，而是：

> **一个正在把三省六部 Agent 系统从概念原型升级为工程化平台的 V2 分支。**

如果你对以下方向感兴趣，这个仓库会值得关注：

- Multi-Agent Orchestration
- Event-Driven Systems
- Agent Observability
- Human-in-the-loop Workflow
- Task Audit / Replay
- 以组织制度设计 AI 协作系统

---

## License

沿用原项目 License，详见 `LICENSE`。
