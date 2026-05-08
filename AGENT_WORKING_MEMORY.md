# IPFAR 项目工作记忆

> 用于 AI 助手跨会话恢复项目上下文。最后更新：2026-05-08

## 仓库
- **IPFAR-specs**: `github.com/LWDJD/IPFAR-specs`
- **IPFAR**（主实现）: `github.com/LWDJD/IPFAR`

## 已完成的规范文档
- `数据结构规范.md` — Tags 标准、元数据 JSON、CAR v2、Reference、PoW、发现流程
- `项目规划.md` — Bitswap、全量验证、缓存、增量DB、网关接口、随机游走等核心条目

## 关键决策记录（2026-05-08）
1. PoW 以 §6.2 为准：< 100 MiB 需 PoW，≥ 100 MiB 无 PoW
2. 去重采用当前 Reference 显式引用方案，放弃原始 Feistel 密码学方案
3. 发现机制双模式：基础 GraphQL + 高级矿工直询
4. 数据结构规范只做数据层设计，其他核心条目放在 `项目规划.md`

## Git 协作规范
- 新分支 → PR → dev → 用户合并后删除分支
- SSH: `~/.ssh/id_ed25519_ipfar`, Host `github.com-ipfar-specs`
