# OpenMAIC 贡献记录 · NVIDIA NIM Provider 与兼容性修复

> 我为 **[THU-MAIC/OpenMAIC](https://github.com/THU-MAIC/OpenMAIC)**（清华大学多智能体课堂）提交的贡献。
> *My contribution to THU-MAIC/OpenMAIC: NVIDIA NIM provider and cross-platform compatibility fixes.*

[![Upstream](https://img.shields.io/badge/upstream-THU--MAIC%2FOpenMAIC-3B7DDD?logo=github&logoColor=white)](https://github.com/THU-MAIC/OpenMAIC)
[![License](https://img.shields.io/badge/upstream%20license-MIT-green.svg)](https://github.com/THU-MAIC/OpenMAIC/blob/main/LICENSE)
[![Extracted](https://img.shields.io/badge/now%20in-llm--provider--kit-7F52FF?logo=npm&logoColor=white)](https://github.com/soyoung5565/llm-provider-kit)

---

## 📖 背景

第一次接触 OpenMAIC 时，我在跑本地部署时撞到四个具体问题。每一个都不是"配置填错了"，
而是**框架本身缺少必要抽象**：

1. **接不了 NVIDIA NIM** —— NIM 上跑 Kimi K3 需要在 provider 表里登记模型元数据，
   而 provider 表是硬编码的，没有扩展点。
2. **muse-spark 模型直接 500** —— 这类模型只在 `/v1/responses` 上可用，
   但框架按"默认 OpenAI 兼容"的假设把请求打到了 `/v1/chat/completions`。
3. **Anthropic 兼容端点 401** —— MiniMax 的 anthropic-compat 端点要求把 API Key
   放在 `X-Api-Key` 头里，而不是标准的 `Authorization: Bearer`。
4. **Windows 上偶发写盘失败** —— `fs.rename` 在目标文件被文件监视器/杀软短暂持有时
   抛 `EPERM`/`EACCES`/`EBUSY`，导致课堂状态文件写不进去。
   另外**批量 TTS 串行生成**拖慢了整节课的生成时间。

## ✨ 我的改动

单个提交 `feat: add NVIDIA NIM provider (Kimi K3) + compat & performance fixes`，
8 个文件，**+96 / −10 行**：

| 文件 | 改动 |
|---|---|
| `lib/ai/providers.ts` | 新增 NVIDIA NIM provider（Kimi K3 模型元数据）；`muse-spark-*` 走 `/v1/responses`；MiniMax 端点改用 `X-Api-Key` 头 |
| `lib/types/provider.ts` | provider 类型系统补全 |
| `lib/server/provider-config.ts` | provider 配置注册 |
| `lib/server/classroom-media-generation.ts` | TTS 改**有界并发**生成（`TTS_CONCURRENCY`，默认 4） |
| `lib/server/classroom-storage.ts` | 原子重命名加 `EPERM/EACCES/EBUSY` 退避重试 |
| `instrumentation.ts` | `process.on` 加 SSR 运行时守卫 |
| `.gitignore` | 忽略本地工作文件 |
| `public/logos/nvidia.svg` | NIM 图标 |

## 🔗 这份贡献现在去哪了

OpenMAIC 是个体量很大的 Next.js 应用（上游 4 万+ star），直接 fork 整个仓库对
"想复用这四个能力"的人没有价值。所以我把它们**重写并抽成了独立库**：

### → **[`soyoung5565/llm-provider-kit`](https://github.com/soyoung5565/llm-provider-kit)**

| 本次贡献 | 独立库中的模块 |
|---|---|
| NIM provider + Kimi K3 元数据 | `providers` —— 数据驱动的 Provider 注册表 |
| muse-spark 走 `/v1/responses` | `providers` —— `shouldUseOpenAIResponsesApi()` 路由谓词 |
| MiniMax `X-Api-Key` 适配 | `compat` —— `buildAnthropicHeaders()` |
| TTS 有界并发 | `concurrency` —— `runWithConcurrency()` |
| Windows 原子写重试 | `fs` —— `writeJsonFileAtomic()` |

独立库版本比原改动多了**44 个测试用例**、完整的 TypeScript 类型定义、
以及可按模块引入的导出结构（`llm-provider-kit/providers` 等）。

## 📄 原始补丁

完整的上游补丁保存在本仓库的 [`contribution/`](./contribution/) 目录：

- `openmaic-nim-contribution.patch` —— 可直接 `git apply` 到 THU-MAIC/OpenMAIC 的 git format-patch 格式补丁
- `openmaic-nim-contribution.diff` —— 同一改动的 unified diff

> 验证方式（在上游仓库根目录）：
> ```bash
> git apply --check contribution/openmaic-nim-contribution.patch
> ```

## 📄 License

本仓库内容（补丁文件与说明文档）以 MIT 发布，与上游 OpenMAIC 的协议一致。
补丁本身是针对 THU-MAIC/OpenMAIC 的贡献，其著作权归原作者所有。
