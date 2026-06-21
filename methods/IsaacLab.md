---
method: IsaacLab
tags: [method, tool, simulation, reinforcement-learning, robotics]
aliases: [Isaac Lab, IsaacLab 3.0]
platform: NVIDIA Isaac Lab
version: v3.0.0-beta2
isaac_sim: "6.0.0"
status: beta
source: 官方 Release Notes + 官方文档（见文末「参考来源」）
reviewed: 2026-06-21
---

# IsaacLab

> [!abstract] 一句话
> **Isaac Lab** 是 NVIDIA 开源的**机器人学习仿真框架**，把强化学习（RL）、示范学习（imitation / learning-from-demos）、运动规划等常见研究工作流统一起来；底层可在 **PhysX / Newton 等多物理后端**上运行，并通过 **Isaac Sim / Omniverse** 提供照片级渲染与大规模 GPU 并行环境。本卡片基于 **v3.0.0-beta2**（2026-06-17 发布）整理。

---

## 适用场景

对本组（[[README|NeuroRobotics Lab]]）而言，IsaacLab 适合：

- **腿足 / 人形 / 机械臂的 RL 训练**：内置 Anymal、Unitree、Franka、UR10、Humanoid、Cartpole 等机器人与 locomotion / manipulation / navigation 任务库，开箱即用。
- **大规模并行采样**：GPU 上向量化数千个并行环境，配合多 GPU / 分布式训练。
- **Sim-to-Real 与策略迁移**：可在快速后端（Newton）上跑训练，再切到照片级 RTX 渲染做感知验证。
- **支持多种 RL / IL 框架**：`rsl_rl`、`rl_games`、`skrl`、`sb3`、`rlinf` 统一入口。

> [!note] 与本组方向的关系
> 神经-机器人方向常用的外骨骼 / 步态控制策略训练，可借 IsaacLab 的 locomotion 任务与并行环境快速搭基线。相关工作沉淀到 [[methods|方法卡片]] 与 [[literature/|文献笔记]] 时，记得回链本卡片。

---

## 核心架构

| 层 | 内容 |
|---|---|
| **物理后端（multi-backend）** | PhysX、Newton、OVPhysX、Isaac RTX、OVRTX，以及 **kit-less** 无 Kit 执行；v3.0 起统一后端选择、场景数据路由与错误报告 |
| **Newton 后端** | GPU 加速、**无需完整 Isaac Sim** 即可跑；本版新增 ray-caster、frame-transformer、IMU/PVA、contact、joint-wrench、deformable、VBD-coupling，并提供四足/双足的 rough-terrain 预设 |
| **渲染 / 相机** | RTX 与基于相机的渲染；新增 `isaaclab_ppisp`（renderer-agnostic 的物理可信图像信号处理管线）、HDR 输出；`TiledCamera` 已并入核心 `Camera` 类 |
| **仿真宿主** | 可**独立用 Newton**，或接 **NVIDIA Isaac Sim + Omniverse** 获得照片级可视化 |
| **训练入口** | 统一的 `train` / `play` 控制台入口，跨 `rsl_rl` / `rl_games` / `skrl` / `sb3` / `rlinf` |

设计哲学：**modular（模块化）、agile（敏捷）、open-source（开源）、batteries-included（开箱即用）**。

---

## v3.0.0-beta2 更新要点

> [!info] 定位
> 官方称这是 **3.0 beta 开发线上的"稳定化与能力补齐"版本**（stabilization & enablement release），不是大改架构，而是把 3.0-beta 的多后端能力夯实、补齐 Newton 工作流、增强渲染。

**新特性 / 改进**
- **多后端稳定化**：PhysX / Newton / OVPhysX / Isaac RTX / OVRTX / kit-less 全线打通，后端选择与不支持组合的报错更清晰。
- **Newton 能力扩展**：补齐多种传感器与耦合（见上表）；新增四足/双足崎岖地形预设；实验性 OVPhysX（kit-less PhysX）支持。
- **渲染**：新增 `isaaclab_ppisp` 物理可信 ISP 管线、HDR 输出，覆盖 Isaac RTX / OVRTX / Newton Warp。
- **安装与开发体验**：实验性根目录 `uv run train` / `uv run play`（从全新 checkout 直接跑）；安装 extras 简化为 `isaacsim` 和 `all` 两项；补充 Docker / CloudXR / teleop / 物理后端文档。
- **训练统一**：`train` / `play` 入口统一；引入**带类型的预设选择器**，如 `physics=physx`、`physics=newton_mjwarp`、`renderer=...`。

**重要 Bug 修复**
- 修复 reset 后 PhysX / Newton 上的传感器数据陈旧问题。
- 修复 manager-based 与 RL 环境的内存泄漏。
- 修复 OVRTX 多 GPU 运行与克隆环境的相机输出。
- 修复 PhysX 场景上的 Newton 可视化更新；修复 `rl_games` / `SB3` 的 checkpoint 回退。

---

## 复现要点（依赖与安装）

> [!example] 关键版本（v3.0.0-beta2）
> - **Isaac Sim**：`6.0.0`
> - **Python**：`>=3.12,<3.13`
> - **PyTorch**：`2.10.0`（x86_64/Windows → CUDA 12.8；aarch64 → CUDA 13.0）
> - **Newton**：`1.2.1` 线；**OVPhysX runtime**：`0.4.13`；**OVRTX**：`0.3.x`
> - 新增依赖 `pyglet>=2.1.6,<3`（Newton GL 视频录制）

最快上手（实验性）：从全新 checkout 直接

```bash
uv run train   # 训练
uv run play    # 回放 / 评估
```

预设示例：`uv run train physics=newton_mjwarp renderer=...`（带类型的预设 token，不再被折叠处理）。

---

## 踩坑 / Breaking Changes

> [!warning] 升级注意
> 本版为 **beta**，**正式 3.0 发布前仍可能有破坏性变更**（官方预计 1–2 个月内发布正式版）。从旧版本迁移时重点检查：
> - **Preset CLI**：移除 `isaaclab_tasks.utils.fold_preset_tokens`；`setup_preset_cli()` 现按原样返回 token。
> - **Teleop Replay**：移除 `isaaclab_teleop.automation` 包，改用 Isaac Teleop 的 MCAP replay（`teleop_replay_agent.py`）。
> - **Deformable APIs**：迁到后端中立的 `isaaclab.assets` / `isaaclab.sim` 导入。
> - **OVRTX 配置**：`use_cloning` → `use_ovrtx_cloning`；移除 `temp_usd_suffix`。
> - **相机 API**：`TiledCamera` 已并入 `Camera` 类。
> - 部分 Newton 特性仍是**后端专属**，跨后端前先确认支持矩阵。

---

## 适用范围（scope）

- 本卡片结论**仅针对 v3.0.0-beta2**；正式 3.0 发布后需复核版本号、依赖与 API。
- 内容来自**官方 Release Notes 与官方文档**，未在本机实测安装；具体环境（CUDA / 驱动 / Isaac Sim 版本）以官方安装文档为准。
- 选 Newton（快、轻、可 kit-less）还是 Isaac Sim + RTX（照片级渲染）取决于任务是否需要高保真感知，按需取舍。

---

## 参考来源

- **Release Notes（v3.0.0-beta2）**：https://github.com/isaac-sim/IsaacLab/releases/tag/v3.0.0-beta2
- **官方文档（该版本）**：https://isaac-sim.github.io/IsaacLab/release/3.0.0-beta2/index.html
- **GitHub 仓库**：https://github.com/isaac-sim/IsaacLab

## 相关论文 / 笔记
- 官方论文（arXiv）：[Isaac Lab](https://arxiv.org/abs/2511.04831) — https://arxiv.org/abs/2511.04831
- [[@ ]] <!-- 之后把基于 IsaacLab 跑过的工作 / 引用 IsaacLab 的论文回链到这里 -->
