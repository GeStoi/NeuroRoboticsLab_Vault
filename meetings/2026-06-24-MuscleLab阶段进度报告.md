---
date: 2026-06-24
attendees: 
topic: MuscleLab 阶段进度报告（学术理论 + 工程实现）
tags: [meeting, progress-report, MuscleLab]
---

# 2026-06-24 · MuscleLab 阶段进度报告

> 本报告承接上一次阶段汇报（2026-06-12《成果总结与阶段报告》），补齐其后 M5 → M6 → M7 → M8 四个阶段的进展，并按要求**明确划分为「学术理论报告」与「工程实现报告」两部分**，分别从科学结论与系统实现两个角度如实反映现阶段进度。所有定量数字均取自 `.omc/` 各阶段 verdict / 结论文档，可溯源、未做修饰。
>
> *命名说明：本项目对外正式命名为 **MuscleLab**；代码仓库目录沿用历史开发代号（`musclemimic_dev`），不代表项目名，亦与作为对照基线的另一项目不应混淆。*

---

## 摘要（一句话定位）

MuscleLab 用**可解释的分层架构**（Stage-A 力矩策略 → Stage-B 肌肉分配）驱动一具 354 块 Hill 肌肉的全身骨骼肌模型模仿人体动作。截至本阶段，主线 M0→M4 已结题，算法层三轮重设计（ver2/ver3/ver4）与三个机制探查阶段（M5/M6/M7）均已结题，**当前活跃方向为 M8（抗扰 Stage-A 力矩层 + 失败挖掘 MoE）**。本阶段最重要的进展是：**把"~60% 全片段存活率天花板"从一个数字，逐层推进为机制清晰、五至八条方法交叉一致的科学结论**——并在 M8 首次定位到 `axial_rotation`（躯干轴向旋转）是唯一的"核心不可行"自由度，且其不可行是策略**过度索求**而非动作本身要求，为下一步"可行性感知奖励"留出了一个尚未被证伪的真实缝隙。

---

# 第一部分 · 学术理论报告

> 本部分回答"我们科学上学到了什么、为什么"。

## 1. 研究问题与科学命题

**研究问题.** 能否用一个**生理可解释**的分层控制器，让一具肌肉骨骼全身模型（而非抽象力矩驱动模型）稳健地模仿人体运动数据？

**核心命题（分层 + 可解释）.** 把控制拆成两层：
- **Stage-A（运动学/力矩层）**：PPO 策略输出 27 维期望关节力矩 `τ_des`，优化模仿奖励（MimicReward）。
- **Stage-B（肌肉层）**：把 `τ_des` 翻译为 354 维肌肉激活 `a∈[0,1]³⁵⁴`，可走在线 QP 最优分配，亦可走学习型 MuscleNN。

**裁定标准（铁律）.** 唯一被承认的成功判据是**闭环 on-policy 全片段存活率**：在 39/108 个 teacher-limited KIT 片段上，从片段起点 rollout，根高 `z < FALL_Z = 0.30` 即判摔。开环重放/探针必然发散、鉴别力为零，绝不用于判存活或可行性。

## 2. 核心科学结论：~60% 天花板及其机制分解

**结论.** "分层 `τ_des` → 肌肉分配"框架 + 不可微 Warp-MJX 平台的全片段存活率天花板约为 **~60%**；要达到 ≥90% 必须跳出该框架（可微仿真重建 / 端到端直驱），这两条路均被刻意界定为本阶段 out-of-scope。该结论已由 **M3 / M4 / ver2 / ver3 / ver4 / M5 / M6 / M7** 多条独立方法路线交叉一致确认。

天花板由若干**可量化、相互独立又彼此印证**的机制复合而成：

| 机制 | 本质 | 决定性证据 |
|---|---|---|
| **FM-B 力锥不可行** | 单步 QP 在低/零/变号力臂的旋转、额状面 DOF 物理无解 | M4 T0：43 个 MuscleNN-fail 片段中 **39 个最优 QP 也 fail**；失败由"旋转 DOF 含量"驱动（r=0.45），与时长无关（r=−0.05）|
| **FM-C 临界稳定脆弱** | 单帧确定性模仿误差指数放大（Simchowitz 下界，arXiv:2503.09722）| 直接杀死 RL：在脆弱肌肉环里做探索，噪声本身即致摔（M3 RL-on-muscle 3 pilot 全 0%）|
| **平台不可微墙** | 不可微 Warp-MJX 挡死一切 look-ahead / 可微 MPC 老师 | M4 T5：梯度 MPC 透 `mjx.step`(Warp) → FFI 不可微；纯 JAX-MJX 被求解器 `while_loop` 挡（修它=平台底层=out-of-scope）；采样 MPC 内层 rollout 不能用 Warp |
| **学习型替代严格劣于精确 QP** | 学习型 MuscleNN 要么塌缩（忽略目标通道）、要么至多近似 QP 且残差在 full-clip 累积致沉 | ver3/ver4：同一 V2-equiv θ，过固定 QP=23.1%、过学习 ψ=**0/39** |
| **regime gap（真对手）** | "RSI 片段训练 alive" ≠ "full-clip 从起点 eval" | M6 定量坐实：29 个确定摔片段**没有一个**即时不可行，全部跑 ≥1.6s（最短 165 步）才摔 |

## 3. 近阶段机制级增量（M5 → M8）

### M5 — 时间窗可行性分配：myopia headroom 真实但不足（CLOSED, 06-16）
- 用户对"held-then-zero 是伪下界"的质疑**成立**：合法的 receding-horizon（每步重规划）确实优于"打一拳撒手"。
- 但即便给在线 CEM oracle 最好条件（kplan=120≈1.2s 前瞻、每步重规划、R=8 取分布）：clip 23 uplift **+16.1pp**、摔点从确定性的 212 步右移到含 600/660 的重尾分布——**短视修正确有 headroom，但 0/8 撑到末**；clip 95 无可复现 uplift（R=3 的"1/3 救活"被 R=8 戳破为噪声）。
- **机制增量**：首次证明**短视确有真实 headroom（硬片段摔点可被前瞻 Δτ 显著右移），但量级不足以根治**——"短视不是天花板主因"。
- 同期独立发现：**native CPU ≠ Warp 平台墙**（M5.0 否决 native-shooting 路线）；**Warp 批内/批间非确定性**（±1 片段），由此确立"信号需 ≥±3 片段、每片段 R≥3 replica 取均值"的统计护栏。

### M6 — ~60% 天花板成因定量分解（CLOSED, 06-16）
- 对 34 个失败片段做 5 因子共现分解：**(a) 参考力矩不可行 ≈0、(c) 平台噪声 5、(d) 短视 ≈0–1、(b) 累积 ⊗ (e) 肌肉层渐沉 = 29/29 耦合主导**。
- **最强单一发现**：全部 29 个确定摔片段都能闭环跑 **≥165 步（1.6s，最长 5.4s）才摔，没有一个起步即摔** → 失败是**累积**而非**根本不能**。
- **机制升级**：把"~60% 天花板"从数字升级为机制结论——**瓶颈在固定 QP 肌肉层的 full-clip 长时程精度保持**，不在瞬时可行性/参考/短视。这定量坐实了 ver3 命名却未量化的 regime gap，并指明：**若要破墙，正确攻击点是 full-clip 精度保持，而非再加 horizon corrector 或换学习型 ψ（二者均已证负）。**

### M7 — Probe-0 共收缩 kill-test：精度两难（CLOSED, 06-17）
- 用 oracle 零空间共收缩做注入实验，配 δ 扫描（δ∈{0,3,8,15,30} N·m）分离"刚度本身有害"与"力矩精度退化有害"：
  - **δ=0（严格保住力矩精度）**：共收缩几乎注不进（co_lift≈0.003），且对存活**零影响**（uplift≈0）；
  - 害处严格随注入后残差膨胀**单调增长**（δ=15 时 6/6 片段存活变差）。
- **机制裁定**：Stage-B 面临**精度两难**——保精度则无共收缩余地；要余地则必损精度，而损精度直接喂给 M6 认定的杀手机制（累积追踪误差→渐沉→摔）。这是 **Herzog-Binding (1993) KKT 定理**（凸 effort 代价 + 力矩等式约束 ⇒ 单关节拮抗共收缩恒为 0）的**闭环定量验证**。
- **影响**：不应建造 "stiffness-aware allocator"（核心前提双向证伪）。**Stage-A balance（正交、有 V2 把肌肉基座从 46%→58% 的先例）是唯一尚未被触及的杠杆** → 触发向 M8 的转向。

### M8 — 抗扰 Stage-A 力矩层 + 失败挖掘 MoE（活跃中, 06-23 起）
M8 不动固定 Stage-B，转而改造 Stage-A 的鲁棒性/可执行性。本阶段已完成 4 个**诊断探针**（P1–P4，零训练，仅 oracle / 单 rollout 测量），用以判断"是否存在可救的缝隙"：

| 探针 | 问题 | 结果 |
|---|---|---|
| **P1 锥裕度** | 哪些 DOF 的 `τ_des` 落在肌肉作动锥外？ | **`axial_rotation` 是唯一核心**：V2 的 τ_des 在锥外占 **23.3%**（near-edge 28.4%）；其余 DOF 全 ≤3.6%。`mtp`（脚趾）虽锥外率次高但锥宽仅 ~11 N·m 且需求极小，属红鲱鱼；ankle 次级。 |
| **P2 轴向逆动力学需求** | 动作本身**要求**多少轴向力矩？ | **参考动作的轴向逆动力学力矩在锥外仅 0.0%**（vs V2 τ_des 的 23.3%）→ **动作不需要不可行的轴向力矩，是 V2 在过度索求**。判定为 **OVER-DEMAND（M8 可处理）**。 |
| **P3 锥内裁剪杠杆** | 把 τ_des 在 QP 前裁进锥内，能否救存活？ | **WEAK/NEGATIVE**（不裁=2.67 > 仅裁轴向=1.33 > 全裁=1.0 存活均值）→ **过度索求不是致命近因**，朴素裁剪反而有害。 |
| **P4 裁剪 noop 对照** | P3 的负向是真扰动还是数值假象？ | 确认裁剪是**真扰动**（改变实现力矩）→ P3 负向为真；但**仅证伪"对冻结策略做后处理裁剪"，未证伪"训练一个可行性感知奖励"**。 |

**M8 当前机制判读（诚实版）**：轴向旋转的不可行是**可由 Stage-A 在分布上学会绕开的过度索求**，而非结构性死锁——这是 M3~M7 从未单独测过的真实缝隙。但朴素的后处理裁剪已证负，**剩余唯一未证伪的假设 = 训练一个"可行性感知 / 平衡感知"的 Stage-A 奖励**，让策略在 motor twin（94.9% 上界、有充分 headroom）里学会少索求不可行的躯干扭矩 + 攻 regime gap（从片段起点训练 + 拉长 episode）。**现实预期标定为 23.1%→~30–40%（乐观逼近 60%），绝不是 90%**——把目标设成 90% 等于换尺子。

## 4. 理论依据与文献支撑

- **Simchowitz et al. (arXiv:2503.09722)** — 平滑/确定/马尔可夫单帧模仿器误差指数放大下界，解释了"63% 不是调参问题、DAgger 改不了策略类"。
- **Herzog & Binding (1993)** — 凸 effort 代价 + 力矩等式约束的 KKT 解结构性零共激活，被 M7 闭环定量验证。
- **Peng et al. (2017)** — PD/MTU 作动优于直接力矩（学得更快、更鲁棒），机制为嵌入式局部反馈（preflex）；启发了 ver4 的 PD/加速度接口设计。
- **Lee et al. (SNU, 2019)** — 可扩展肌肉作动控制；其"上层发 PD/加速度靶 + 两层早期 co-adapt + 单片段定性评测"与本项目"发刚性 τ_des + 暖启理想力矩平台 + 39 full-clip 硬存活 KPI"的**问题设定差异**，解释了为何其架构在本项目复刻不出。
- **Delp et al. (1999)** — 旋转 DOF 的低/零/变号力臂，是 FM-B 力锥不可行的解剖学根据。

## 5. 学术贡献定位（正向 + 负向，等价值）

- **正向贡献**：一套**可解释的分层骨骼肌运动模仿框架**，在 354 肌肉全身模型上以监督 + DAgger 达到 **63% 全片段闭环存活**，且推理较在线 QP 快 **~40×**。
- **负向贡献（同等价值，且更稀缺）**：用五至八条独立方法路线、逐层机制证据（FM-B 力锥不可行 + FM-C Simchowitz 临界脆弱 + 平台不可微墙 + 学习型替代严格劣于精确 QP + M6 定量化的 full-clip 累积漂移主导），**严格刻画出"为何 ~60% 是该框架+平台的天花板"**——为后续工作明确了"跳出框架"是突破 ≥90% 的唯一已知路径。这是一份机制清晰、可发表的负向结果。

---

# 第二部分 · 工程实现报告

> 本部分回答"我们造了什么系统、工程上如何保证结论可信"。

## 1. 系统架构总览

核心框架包是一个基于 loco_mujoco 衍生的通用分层 RL 训练/仿真平台（67 个 Python 源文件，经 Understand-Anything 知识图谱分析为 7 层、68 文件、111 函数、72 类）；项目真正的**研究增量**（Stage-B 的 QP / MuscleNN / 联合 trainer / 诊断器）以脚本形态沉淀在 `.omc/` 各里程碑目录。

```
实验编排层 runner/ ──► 训练算法层 algorithms/(PPO + JAX-scan) ──┐
                       环境与模型层 environments/(LocoEnv·MyoFullBody 354 肌肉·motor 孪生 27 力矩)
                                              │
                       仿真核心层 core/ ◄──────┘
                         mujoco_mjx.py(MJX/Warp 闭环步进)· reward/trajectory_based.py(MimicReward)
                       RL 基础设施 rl_core/ · 工具 utils/(AMASS retarget) · 可视化 viewer/(Viser)
```

**一次训练/评测数据流**：`runner/engine.py`(装配) → `algorithms/ppo/runner.py`(JAX-scan 训练循环) → 策略-价值网络(输出 27 维 τ_des) → `environments/humanoids/myofullbody.py`(模型) → `core/mujoco_mjx.py`(物理步进) + `core/reward/trajectory_based.py`(MimicReward) → `core/wrappers/mjx.py`(obs/reward/done) → `runner/eval_utils.py`(闭环存活评测) → `viewer/`(渲染)。

**模型与数据基座（M0）**：MuJoCo MyoFullBody（`nq=89 / nv=88 / nu=354`），27 个可驱动 DOF + 6-DOF 浮动根，51 个 equality 约束；motor 孪生模型（`nu=27`，纯关节力矩）与肌肉模型运动学同构，保证 `τ_des` 可精确转移。数据为 AMASS 经 GMR retarget 到 MyoFullBody（KIT / Transitions），train ~972–1082 clip、**test 108 clip**。

## 2. 关键组件实现

- **Stage-A PPO 力矩策略（M1）**：`ResidualFCNet`，MJX/Warp 后端。armfix 修复手臂翻转根因（rquat 权重 0.01→0.3 + qpos 0.1→0.3）；A.2 归一化版本权重 ÷1.6、from-scratch 2.048B 步 → `checkpoint_12500`。关键工程发现：**运动学持平 ≠ 肌肉闭环持平**（A.2 norm 运动学好但在肌肉环里更脆）。
- **Stage-B QP 分配器（M2）**：利用 Hill 力对激活的**精确仿射**关系 `F = gain(q,q̇)⊙a + bias`（残差 4.5e-13）→ 降维 box-QP `min‖S·F(a)−τ_des‖² + λ‖a‖`（λ=1e3，FISTA ~300 步）。计算力矩闭环、Warp、SUBSTEPS=5（500Hz）。实测 `E_QP=0.150`、`L_QP=1.03ms`、存活 60–61%。
- **学习型 MuscleNN + DAgger（M3）**：学 `a* = f(q,q̇,τ_des)`（191→512³→354，sigmoid）替代 QP；监督蒸馏 MSE ↓2.3 个数量级、推理 ~40× 快于 QP；DAgger 在线聚合（QP 标注）治愈分布漂移，A.1+DAgger r5 达 63.0%。
- **闭环双网联合 trainer（ver3/ver4）**：`v3_joint_trainer.py` / `v4_joint_trainer.py`，含 build-gate（ver4 21/21，Warp M⁻¹ densify 验 EoM rel-err 3e-6）、criterion-gated 课程、reset_critic_head、固定 QP allocator、全套 env-var 杠杆。ver4 经诊断坐实"学习型 ψ 塌缩"（q̈_d→a relstd **0.007 vs 固定 QP 0.91**，衰减 130×）+ critic 坏死，为机制证伪提供铁证。

## 3. 评测与诊断基建（结论可信度的核心）

- **闭环 full-clip eval harness**：`m3_fix_ver2/c7_eval_v3.py`（39-clip teacher-limited 集、FALL_Z=0.30、per-clip JSON）。
- **正确的 θ+ψ 闭环 eval**：`m3_fix_ver3/v3_eval.py` — 任何学习型肌肉层方案必须用它（绝不用 `c7_eval_v3` 评学习型 ψ：后者用固定 QP 评 θ、从不加载 ψ，曾造成"学习 MuscleNN≈20.5%"的假象）。
- **平台非确定性护栏**：每片段 R≥3 replica（不同 seed）取 survival fraction 均值，信号阈值 ≥±3 clip（±1 clip 为噪声），强制 `nconmax ≥ 102`。
- **信号诊断器**：`m3_fix_ver4/diagnose_v4_signal.py`（value/advantage 健康度 + 逐层动作敏感度 + QP 对照），是 ver4 裁定的产出工具。
- **M8 诊断探针**：`m8/m8_p1_conemargin.py`（逐 DOF 锥裕度）、`m8_p2_axial_idemand.py`（参考逆动力学需求，`mj_inverse`）、`m8_p3_axial_clip.py`（锥内裁剪 × R replica）、`m8_p4_clip_noop.py`（同状态额外 QP 求解测量裁剪扰动，排除发散混淆）。

## 4. 当前在建工程（M8 MVP）

M8 的工程策略是**最大化复用、局部新增**（已确认复用点：`resume_from` 暖启、`adaptive_sampling` hard-clip 加权雏形、`runner.py` 课程门控机制、`c7_muscle_loop_env.py` 闭环 QP 集成、`qp_allocator.py`）。需新增（均为附加，非重构）：

1. **disturbance 模块**（最大新代码，greenfield 但局部）：在 motor twin 的 `mjx_step` / wrapper 注入 **P0 两项扰动**——按 DOF 带系统偏置的 `τ_des` 残差注入（必须**校准自真实 QP 残差统计**）+ reset-state 扰动（直接攻 regime gap），配置驱动 + curriculum scale（从 0 起、按 term_rate 门控增长）。
2. **QP 残差校准脚本**：跑 V2 过 QP，logging per-DOF/per-phase `tau_real − tau_des` 分布。
3. **eval 诊断扩展**：`c7_eval_v3.py` per-clip JSON 加 `fall_step / z_drift_slope / pelvis_roll·pitch / qp_residual_per_dof[27] / slack_rate` 等字段 + R-replica 包装。
4. **mining/clustering 脚本**（离线）：按失败模式分簇（簇 A 结构性不可行→排除；簇 B 晚期漂移→MoE 主目标；簇 C 接触/失衡→次目标）。**关键决策门**：若簇 B/C 几乎为空（失败被簇 A 主导），则 MoE 无意义，M8 以"从新角度坐实天花板"收口。
5. **clip-id 路由 eval**：评测时按 clip 分派 {generalist, specialist}（v1 用零运行时风险的 clip-id 路由 + 专家内部用有界残差 Δτ）。

**注**：当前 P3 朴素裁剪已证负，故 M8.1 的"训练可行性感知奖励 + 立即过 QP 闭环 eval 验迁移"是接下来的关键判别步；按全量训练红线，需报告并取得显式放行后方可启动。

## 5. 可复用资产清单

| 资产 | 路径 | 用途 |
|---|---|---|
| force-space QP | `m2/qp_allocator.py` | `solve_qp_fista`，可微、已 vmap |
| 闭环双网 trainer | `m3_fix_ver4/v4_joint_trainer.py` | build-gate 21/21、全套杠杆 |
| 正确 θ+ψ eval | `m3_fix_ver3/v3_eval.py` | 学习型肌肉层的唯一合法存活裁定 |
| 信号诊断器 | `m3_fix_ver4/diagnose_v4_signal.py` | value/adv + 敏感度 + QP 对照 |
| 闭环 QP 环境 | `m3_fix_ver2/c7_muscle_loop_env.py` | action→τ_des→QP→activations→step |
| balance reward | `core/reward/trajectory_based.py`(v3_balance) | M7.1/M8 平衡奖励项 |
| 知识图谱 | `musclemimic/.understand-anything/knowledge-graph.json` | 代码结构导览 |

## 6. 工程方法学教训（反复出现，已沉淀为铁律）

1. **eval-harness 校准是头号混淆源** — ver2 的 `CKPT_DEFAULT→armfix` 假 0/39；ver3 的 `c7_eval` 用 QP 从不加载 ψ 的假 20.5%。**任何存活数字先验证 eval 加载的是正确的策略/下层。**
2. **闭环 on-policy 是唯一裁定** — 开环探针零鉴别力（反复纠正）。
3. **instrumentation 优先** — "ep_length=0 没在学"曾被证实是 `num_done==0` 窗口假象误诊；先量清楚再下结论。
4. **平台非确定性必须统计处理** — Warp ±1 clip 非确定，单次二值结果不可信，用 R-replica 均值 + ±3 clip 带。
5. **作者自评绿不可信** — 主线独立复跑屡次抓出 executor self-green 漏掉的 build-breaker；每轮对照安装源码（`hasattr` 而非信文档）。

## 7. 平台与 scope 约束（铁律）

- **平台**：训练/仿真用 **Warp 后端**（肌肉模型在 MJX-GPU `mjx.step` 触发 cuSolver 错误）；eval 用 MuJoCo CPU。
- **算法层 scope**：不改 MJX/Warp 平台底层（求解器）；解法只在算法/模型层，平台级改动一律标记 out-of-scope。
- **全量训练红线**：smoke 默认可跑但须报告；全量/大训练须显式放行。
- **A.1 / A.2 隔离**（A.1=非归一化 dev 基座，A.2-final=归一化验收基准，绝不混表）+ 不就地改 frozen/历史版本文件。

---

## 当前状态与下一步

**当前活跃**：M8 — feasibility-aware / balance-aware Stage-A 奖励（教 Stage-A 别索求不可行的轴向扭矩 + 攻 regime gap）。

**已 banking 的基线与最佳交付物**：

| 交付物 | 存活 | 说明 |
|---|---|---|
| A.1 + DAgger r5（全局最佳） | **63.0%** | 首次肌肉层超过其 QP 老师 |
| A.1 监督 MuscleNN | 56.5% | ≈QP 忠实克隆 |
| V2(pose-tight) + 固定 QP（最佳干净归一化基座） | **58.3%** | 推荐 Stage-B τ_des 源 |
| V2-equiv θ + 固定 QP（39-clip 硬子集基线） | **23.1% (9/39)** | M8 的攻坚基线 |
| V2 力矩平台（无肌肉，上界参照） | **94.9% (37/39)** | 证明瓶颈全在肌肉层 |

**下一步候选（按当前判读优先级）**：
1. **M8.1**：实现 disturbance 模块（P0 两项扰动，τ_des 残差**校准自真实 QP 残差**）+ clip-start 课程，训练抗扰 generalist，**立即过 QP 闭环 eval 验迁移**（需先报告取得放行）。
2. **M8.2**：失败挖掘分簇，强制做可救性判定；若失败被结构性簇主导，则按预案收口为负结果。
3. 仅当存在可分离的"晚期漂移簇"时，才进入 **M8.3** 有界残差专家（clip-id 路由）。

**Action Items**
- [ ] M8.1 disturbance 模块实现 + QP 残差校准脚本 —— 待放行后启动
- [ ] M8.1 抗扰 generalist 训练后立即过 full-clip QP eval（R≥3）验迁移
- [ ] M8.2 失败分簇 + 可救性门控判定

## 相关链接（内部工件）

- 上一次阶段报告：`.omc/paper/2026-06-12_成果总结与展示/musclemimic_成果总结与阶段报告.md`
- M5 结论：`.omc/m5_horizon_feasibility_allocation/M5_2E_CONCLUSION.md`
- M6 成因分解：`.omc/m6_ceiling_decomposition/M6_DECOMP_REPORT.md`
- M7 共收缩裁定：`.omc/m7/PROBE0_CONCLUSION.md`
- M8 计划：`.omc/plans/M8_disturbance_robust_moe.md`；探针 verdict：`.omc/m8/m8_p{1,2,3,4}_*_verdict.json`
- 项目总章程：`.omc/plans/musclemimic-PROJECT-CHARTER.md`

---

*本报告如实反映截至 2026-06-24 的进度；所有定量结论均可溯源至上列 `.omc/` 工件，未做夸大或修饰。*
