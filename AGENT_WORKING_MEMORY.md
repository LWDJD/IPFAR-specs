# IPFAR 项目工作记忆

> 用于 AI 助手跨会话恢复项目上下文。最后更新：2026-05-09

## 项目结构

### 仓库
- **IPFAR**（主实现）：`github.com/LWDJD/IPFAR`（Go 语言）
- **IPFAR-specs**（规范文档）：`github.com/LWDJD/IPFAR-specs`

### 已完成代码模块
- CLI 框架：`internal/flags/`（自定义参数解析，不支持驼峰命名检测）
- 子命令系统：`internal/cmd/`（多级命令支持）
- 日志系统：`internal/log/`（分级日志，文件输出，UTF-8 BOM）
- 配置管理：`config/`（koanf 实现）
- 多语言支持：gotext

### 选型库（已选未实现）
- CAR 解析：`ipld/go-car`
- Bitswap：`ipfs/boxo`
- P2P：`libp2p/go-libp2p`
- DHT：`libp2p/go-libp2p-kad-dht`
- ANS-104 Bundle：`ar-io/arbundles` 或 `permadao/goar`
- KV 存储：`dgraph-io/badger`

## 规范文档状态

### `数据结构规范.md`（已完成，仅四章）

当前分支：`feat/configurable-verification`

- 一、Arweave Transaction Tags（Protocol、Content-Type、IPFAR-Type、Root-CID、Data-TXID、Data-Size、Reference）
- 二、元数据 JSON 格式（version、method、root_cid、data_txid、data_height、data_size、reference、content_type、original_name、pow、pow_alg）
- 三、CAR v2 文件格式（基本要求、上传方式 `raw/bundle`、分块机制、验证要求）
- 四、引用 Reference（设计目标、格式、分块方式、引用粒度、传输与本地格式、链式解析、限制）

> 原五~八章（PoW、防垃圾、重复CID、发现流程）已整合入 `项目规划.md`，不再独立存在于本文件。

### `项目规划.md`（已整合，七章 + 子节）

当前分支：`feat/configurable-verification`

- **一、数据格式** — 引用 `数据结构规范.md` 的 Tag/JSON/CAR/Reference 定义
- **二、验证策略** — PoW 算法详细规范（§2.1：算法 argon2id-light-v1、难度曲线、验证流程）
- **三、发现与获取** — §3.1 GraphQL 模式、§3.2 矿工直询模式
- **四、桥节点行为** — §4.1 可配置验证选项（原 §8.3，含验证项清单、安全等级预设、验证流程图）
- **五、去重与存储** — §5.1 重复 CID 处理（发现规则 + 元数据集合）
- **六、安全与经济** — §6.1 存储费与经济缓解（原防垃圾章节）
- **七、可能相关的内容** — 外部参考链接

### 章节归属对照（原数据结构规范五~八章 → 项目规划）

| 原章节 | 内容 | 归入位置 |
|--------|------|---------|
| 五、PoW | 算法/难度/验证 | 二、验证策略 §2.1 |
| 六、防垃圾 | 存储费+PoW分层 | 六、安全与经济 §6.1 |
| 七、重复CID | 去重规则/元数据集合 | 五、去重与存储 §5.1 |
| 八、发现流程 | GraphQL/矿工直询 | 三、发现与获取 §3.1/§3.2 |
| 八.3 可配置验证 | 验证开关/安全等级 | 四、桥节点行为 §4.1（此前已移入） |

## Pro 对比分析（2026-05-08）

### 覆盖度：12 条规划条目中仅 3 条完全覆盖

### P0 必须修
- §5.3 与 §6.2 **PoW 阈值自相矛盾** — 已于本次重构统一为 100 MiB
- 去重从 Feistel 密码学改为显式 Reference，**路线转变无决策记录**

### P1 高优补
- DHT 发布与 IPNI 发布策略完全缺失
- 5 个核心条目无承接文档：Bitswap、全量验证、本地缓存、增量DB、网关接口

### P2 后续
- 发现机制退化：矿工直询/随机游走/bitlist 全部丢失
- 阈值单位从 MB 变为 MiB 未注明 — 本次重构已统一使用 MiB

详见对话记录全文，下回迭代优先修 P0。

### 待办（用户说"回头再改"）
- ⬜ 用户 review 后合并 PR
- ⬜ 可能需要调整的细节：
  - PoW 算法是否需要进一步简化
  - 字段命名是否需要调整
  - discovery 流程是否需要补充

## 桥协议设计原则

1. **API 来源安全由用户负责** — 用户可配置任意 API（公共网关/自建全节点/网关路由器）
2. **最终数据由 IPFS CID 兜底** — hash(data) == CID 是密码学强制验证
3. **不能牺牲安全性换生态**（与 Arweave 官方的区别）
4. **规范先行** — 先定规范再实现，利于生态复现

## 数据结构关键设计

- 元数据单独交易（`Content-Type: application/json` + `IPFAR-Type: meta`）
- CAR v2 必须含 Index，否则拒绝
- 引用格式 `{txid: {height, cids[]}}` → 同时承担去重和分块
- PoW：Argon2id（1MB/1/1），难度与文件大小成反比
- 分块：大 DAG 拆多个 CAR v2，通过 reference 关联

## 当前工作状态

| 项目 | 状态 |
|------|------|
| IPFAR 代码 | CLI/配置/日志完成，桥核心逻辑未开始 |
| IPFAR-specs | `feat/configurable-verification` 分支，已完成文档整合重构 |
| arweave-light | v0.3.9 MVP 暂停，等待官方 Merkle 证明 API |

## Git 协作规范
- 每次推送创建**新分支**（如 `feat/xxx`、`fix/xxx`、`docs/xxx`），不复用旧分支名
- 用户合并 PR 后删除远端分支
- 本地同步：`git fetch origin --prune && git reset --hard origin/dev`
- SSH: `~/.ssh/id_ed25519_ipfar`，Host `github.com-ipfar-specs`
