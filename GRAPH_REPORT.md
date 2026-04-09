# virtuoso-bridge-lite: Python Bridge for Cadence Virtuoso

**GitHub**: https://github.com/Arcadia-1/virtuoso-bridge-lite
**文档**: https://virtuoso-bridge.tokenzhang.com
**作者**: TokenZhang (Arcadia-1)
**生成时间**: 2026-04-09

---

## 摘要

virtuoso-bridge-lite 是一个 Python 桥接工具，用于控制 Cadence Virtuoso。支持本地和远程执行、Layout/Schematic 编辑、Spectre 仿真。专为 AI Agent 设计，CLI 优先。

---

## 知识图谱摘要

- **节点**: 33
- **边**: 41
- **社区**: 6

---

## God Nodes (最高度数节点)

| 节点 | 度数 | 类型 |
|------|------|------|
| virtuoso-bridge-lite | 15 | 系统 |
| VirtuosoClient | 12 | 类 |
| SchematicEditor | 8 | 类 |
| SKILL IPC | 8 | 机制 |
| LayoutEditor | 7 | 类 |

---

## 核心架构

```
┌─────────────────────────────────────────────────────┐
│                    AI Agent                          │
│                 (Claude Code, Cursor)                │
└─────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────┐
│                  virtuoso-bridge-lite                │
│  ┌─────────────┐ ┌──────────────┐ ┌──────────────┐  │
│  │VirtuosoClient│ │ SSHClient  │ │SpectreSim   │  │
│  └─────────────┘ └──────────────┘ └──────────────┘  │
└─────────────────────────────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Local Mode │  │ Remote Mode │  │ SKILL IPC   │
│             │  │ (SSH Tunnel)│  │             │
└─────────────┘  └─────────────┘  └─────────────┘
          │               │               │
          └───────────────┼───────────────┘
                          ▼
┌─────────────────────────────────────────────────────┐
│                   Cadence Virtuoso                   │
│  ┌─────────────┐ ┌─────────────┐ ┌──────────────┐  │
│  │  Schematic  │  │   Layout    │  │ ADE Maestro │  │
│  └─────────────┘ └─────────────┘ └──────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## 社区 (Communities)

### 🔵 Core Components

**核心架构组件**

| 组件 | 说明 |
|------|------|
| virtuoso-bridge-lite | 主系统 |
| VirtuosoClient | Python 客户端 |
| SKILL IPC | 进程间通信 |
| ramic_bridge.il | SKILL 桥接脚本 |

---

### 🟢 Remote Access

**远程访问功能**

| 功能 | 说明 |
|------|------|
| SSHClient | SSH 客户端 |
| SSH Tunnel | 持久隧道 |
| Jump Host | 跳板机支持 |
| Auto Reconnect | 自动重连 |

---

### 🟠 Schematic API

**原理图编辑 API**

| API | 说明 | 需要坐标? |
|-----|------|----------|
| `add_instance(lib, cell, xy, orient)` | 放置实例 | ✅ 需要 |
| `add_wire(points)` | 画线 | ✅ 需要 |
| `add_net_label_to_instance_term(inst, term, net)` | 标签终端 | ❌ 自动 |
| `add_wire_between_instance_terms(i1, t1, i2, t2)` | 连接终端 | ❌ 自动 |
| `add_pin_to_instance_term(inst, term, pin)` | 创建引脚 | ❌ 自动 |

---

### 🟣 Layout API

**版图编辑 API**

| API | 说明 |
|-----|------|
| `add_polygon(layer, points)` | 多边形 |
| `add_via(layer, xy, rows, cols)` | 通孔 |
| `bus_routing(start_points, end_points)` | 总线布线 |
| `read_layout()` | 读取版图 |

---

### 🔴 Simulation

**Spectre 仿真**

| 分析类型 | 说明 |
|---------|------|
| Transient | 瞬态分析 |
| DC+AC | 直流+交流 |
| PSS+Pnoise | 周期稳态+噪声 |

**结果解析**:
- PSF Parser - 解析 Spectre 输出

---

### 🟡 ADE Integration

**ADE 集成**

| 工具 | 函数 |
|------|------|
| ADE Maestro | `maeCreateTest()`, `maeRunSimulation()` |
| ADE Explorer | `sevRun()` |

---

## 关键关系

| 源 | 关系 | 目标 |
|----|------|------|
| VirtuosoClient | uses | SSHClient |
| VirtuosoClient | communicates_via | SKILL IPC |
| SchematicEditor | provides | add_instance() |
| SpectreSimulator | runs | Transient Analysis |
| virtuoso-bridge-lite | designed_for | AI Agent |

---

## 与 skillbridge 对比

| 功能 | virtuoso-bridge-lite | skillbridge |
|------|---------------------|-------------|
| 本地模式 | ✅ | ✅ |
| 远程执行 | ✅ SSH 隧道 | ❌ |
| 加载 .il 文件 | ✅ | ❌ |
| Layout API | ✅ | ❌ |
| Schematic API | ✅ | ❌ |
| Spectre 仿真 | ✅ | ❌ |
| ADE 集成 | ✅ | ❌ |
| AI Agent 支持 | ✅ | ❌ |

---

## 使用示例

### 连接

```python
from virtuoso_bridge import VirtuosoClient

# 本地
client = VirtuosoClient.local(port=65432)

# 远程
client = VirtuosoClient.from_env()  # 读取 .env
```

### Schematic 编辑

```python
with client.schematic.edit("myLib", "myCell") as sch:
    # 添加实例
    sch.add_instance("gpdk090", "nmos1v", (0, 0), "R0", name="M1")
    sch.add_instance("gpdk090", "pmos1v", (2, 0), "R0", name="M2")
    
    # 自动连线（无需坐标！）
    sch.add_net_label_to_instance_term("M1", "D", "DRAIN")
    sch.add_wire_between_instance_terms("M1", "D", "M2", "S")
    sch.add_pin_to_instance_term("M1", "G", "VIN")
```

### SKILL 执行

```python
# 执行 SKILL 表达式
result = client.execute_skill("1+2")
print(result.output)  # "3"

# 加载 .il 文件
client.load_il(Path("my_script.il"))
```

### ADE Maestro

```python
# 创建测试
client.execute_skill('maeCreateTest("TRAN" ?lib "lib" ?cell "cell")')

# 配置分析
client.execute_skill('maeSetAnalysis("TRAN" "tran" ?enable t ?options `(("stop" "10u")))')

# 运行仿真
client.execute_skill('maeRunSimulation()')
```

---

## 参考文档

- 官方网站: https://virtuoso-bridge.tokenzhang.com
- GitHub: https://github.com/Arcadia-1/virtuoso-bridge-lite
- AGENTS.md: AI Agent 设置指南
- skills/: Agent 技能文件

---

*Generated by knowledge graph analysis*