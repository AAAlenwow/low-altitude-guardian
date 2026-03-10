# low-altitude-guardian

> 低空设备危机应急响应技能包 — OpenClaw / ClawHub Skill

[![ClawHub](https://img.shields.io/badge/ClawHub-low--altitude--guardian-blue)](https://clawhub.com/skills/low-altitude-guardian)
[![Stage](https://img.shields.io/badge/stage-alpha-red)]()
[![License](https://img.shields.io/badge/license-MIT-green)]()

面向低空经济场景的无人设备（无人机、eVTOL、无人配送车等）突发危机应急响应技能包。通过 **感知 → 记录 → 分析 → 匹配 → 决策 → 执行 → 复盘** 的完整闭环，帮助设备在遇到突发情况时自主选择**最优最小损失**规避方案，并持续迭代自身的危机处理能力。

---

## 核心设计

### 损失优先级金字塔

所有决策严格遵守不可逆转的优先级排序：

```
┌─────────────────┐
│  P0: 人员安全    │  ← 绝对最高，不可妥协
├─────────────────┤
│  P1: 公共安全    │  ← 公共设施、交通
├─────────────────┤
│  P2: 第三方财产  │  ← 他人车辆、农作物
├─────────────────┤
│  P3: 本机安全    │  ← 设备自身
├─────────────────┤
│  P4: 任务完成    │  ← 原定任务
└─────────────────┘
```

**硬性否决规则**：人员安全评分 < 80 的方案**无条件淘汰**，无论总分多高。

### 7 阶段闭环

| 阶段 | 说明 | 脚本 |
|------|------|------|
| Phase 1 | 态势感知与情况记录 | `situation_awareness.py` |
| Phase 2 | 危机分级(L1~L5)与分类 | `crisis_engine.py --classify` |
| Phase 3 | 方案匹配 + 最优评分 | `decision_manager.py --match` |
| Phase 4 | 执行监控与动态调整 | `crisis_engine.py --monitor` |
| Phase 5 | 事件记录与分级上报 | `incident_reporter.py` |
| Phase 6 | 知识库自迭代学习 | `crisis_engine.py --learn` |
| Phase 7 | 特殊场景处理 | 完全失控 / 多机协同 / 法规合规 |

### 危机等级体系

| 等级 | 名称 | 响应时限 | 人工介入 |
|------|------|---------|---------|
| **L5** | 灾难性 | < 3秒 | 自主执行，事后通知 |
| **L4** | 严重 | < 10秒 | 自主执行，同步通知 |
| **L3** | 重大 | < 30秒 | 推荐方案，5秒超时自动执行 |
| **L2** | 一般 | < 2分钟 | 等待操作员确认 |
| **L1** | 注意 | < 5分钟 | 仅提醒，操作员决策 |

### 决策评分公式

```
Score = 0.40×S0(人员) + 0.25×S1(公共) + 0.15×S2(财产) + 0.12×S3(本机) + 0.08×S4(任务)
```

---

## 快速开始

### 安装

```bash
# 通过 ClawHub 安装
clawhub install low-altitude-guardian

# 或 clone 本仓库
git clone https://github.com/AAAlenwow/low-altitude-guardian.git
```

### 运行演示

```bash
# 完整危机响应演示（单电机失效场景）
python3 scripts/crisis_engine.py --demo

# 事件报告演示
python3 scripts/incident_reporter.py --demo

# 查看所有解决方案模板
python3 scripts/decision_manager.py --list-templates
```

### 完整流程使用

```bash
# 1. 生成态势快照
python3 scripts/situation_awareness.py --snapshot --trigger "左前电机异常振动"

# 2. 危机分级
python3 scripts/crisis_engine.py --classify --input snapshot.json

# 3. 方案匹配
python3 scripts/decision_manager.py --match --input snapshot.json

# 4. 多方案对比
python3 scripts/decision_manager.py --compare --input snapshot.json

# 5. 生成事件报告
python3 scripts/incident_reporter.py --generate-report --incident-id INC-xxx

# 6. 从反馈中学习
python3 scripts/crisis_engine.py --learn --feedback feedback.json
```

---

## 预置解决方案

| 模板 | 危机类型 | 适用等级 | 历史成功率 |
|------|---------|---------|-----------|
| 单电机失效 | `power_failure.single_motor_loss` | L3 | 92% |
| 全动力丧失 | `power_failure.total_power_loss` | L4-L5 | 65% |
| GPS 信号丢失 | `navigation_failure.gps_loss` | L2-L3 | 88% |
| 通信链路断开 | `communication_failure.link_lost` | L2-L3 | 95% |
| 极端天气 | `environment_threat.severe_weather` | L2-L4 | 90% |

知识库无匹配时自动启动**应急推理**（分解子问题 + 组合策略 + 第一性原理），新方案验证有效后自动入库。

---

## 项目结构

```
low-altitude-guardian/
├── SKILL.md                           # 技能定义（ClawHub 入口）
├── scripts/
│   ├── crisis_engine.py               # 核心引擎（分级/匹配/监控/学习）
│   ├── situation_awareness.py         # 态势感知
│   ├── decision_manager.py            # 决策管理
│   └── incident_reporter.py           # 事件记录与上报
├── assets/
│   ├── solution_templates/            # 解决方案知识库（5个预置模板）
│   └── device_profiles/               # 设备类型配置
├── references/
│   ├── crisis_taxonomy.md             # 危机分类学（6大类 30+ 子类）
│   └── decision_priority_matrix.md    # 决策优先级矩阵
└── tests/                             # 测试
```

---

## 支持的设备类型

| 设备 | 特有场景 | 特殊考虑 |
|------|---------|---------|
| 多旋翼无人机 | 电机失效、螺旋桨折断 | 可悬停，降落选项多 |
| 固定翼无人机 | 失速、发动机停车 | 不可悬停，需滑翔降落 |
| eVTOL | 过渡飞行段故障 | 载人，P0 最高优先 |
| 无人配送车 | 路障、行人冲突 | 低速，停车为首选 |
| 无人船 | 碰撞、进水 | 水域环境 |
| 巡检机器人 | 高空坠落、卡死 | 封闭环境 |

---

## 依赖

- Python 3.8+
- 无外部依赖（纯标准库实现）

---

## 安全原则

- **宁可误报不可漏报** — 过度反应优于低估风险
- **保守决策** — 信息不完整时假设最坏情况
- **可解释性** — 每个决策有清晰推理链路
- **人命无价** — 涉及人员安全的场景零容忍风险

---

## 阶段

**Alpha** — 概念验证阶段，不可用于真实飞行控制。仅作为危机决策辅助分析工具。
