# Virtuoso Bridge Lite Project Specification

**Version**: 2.0  
**Date**: 2026-04-16  
**Status**: Official Reference Document  
**Source**: Knowledge Graph (33 nodes, 41 edges)

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [System Architecture](#2-system-architecture)
3. [Core Components](#3-core-components)
4. [VirtuosoClient API](#4-virtuosoclient-api)
5. [SchematicEditor API](#5-schematiceditor-api)
6. [LayoutEditor API](#6-layouteditor-api)
7. [Remote Execution](#7-remote-execution)
8. [SKILL IPC Protocol](#8-skill-ipc-protocol)
9. [Spectre Simulation](#9-spectre-simulation)
10. [Configuration System](#10-configuration-system)
11. [Best Practices](#11-best-practices)
12. [Security Framework](#12-security-framework)
13. [Error Handling](#13-error-handling)
14. [Integration Points](#14-integration-points)
15. [Development Guidelines](#15-development-guidelines)
16. [API Reference](#16-api-reference)
17. [Examples](#17-examples)
18. [Appendices](#18-appendices)

---

## 1. Executive Summary

### 1.1 Product Overview

**Virtuoso Bridge Lite** is a Python bridge tool for controlling Cadence Virtuoso, designed specifically for AI Agents. It provides CLI-first interface for schematic/layout editing and Spectre simulation.

**GitHub**: https://github.com/Arcadia-1/virtuoso-bridge-lite  
**Documentation**: https://virtuoso-bridge.tokenzhang.com  
**Author**: TokenZhang (Arcadia-1)

### 1.2 Key Metrics

| Metric | Value |
|--------|-------|
| Total Nodes | 33 |
| Total Edges | 41 |
| Communities | 6 |
| API Methods | 10 |
| Classes | 5 |

### 1.3 Core Features

- ✅ Local and Remote Execution
- ✅ Schematic Editor API
- ✅ Layout Editor API
- ✅ Spectre Simulation Support
- ✅ SKILL IPC Protocol
- ✅ SSH Tunnel Support
- ✅ Jump Host Support

### 1.4 Target Users

- AI Agents (Claude Code, Cursor, Copilot)
- EDA Engineers
- IC Designers
- Analog/Mixed-Signal Engineers

---

## 2. System Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      AI Agent Layer                          │
│        (Claude Code, Cursor, Copilot, OpenCode)              │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                 Virtuoso Bridge Lite                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │VirtuosoClient│  │ SSHClient    │  │ SpectreSimulator │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │SchematicEdit │  │ LayoutEditor │  │ SKILL IPC        │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   Local Mode    │ │  Remote Mode    │ │   SKILL IPC     │
│  (Same Host)    │ │ (SSH Tunnel)    │ │ (SKILL Scripts) │
└─────────────────┘ └─────────────────┘ └─────────────────┘
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    Cadence Virtuoso                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Schematic   │  │    Layout     │  │   ADE Maestro    │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   ADE Explorer│  │   Spectre    │  │  Ocean/SKILL     │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Execution Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| Local | Direct connection on same host | Development workstation |
| Remote | SSH tunnel to remote server | Server-based EDA |
| SKILL IPC | SKILL script execution | Legacy integration |

---

## 3. Core Components

### 3.1 Component Registry

| Component | Type | Description | Dependencies |
|-----------|------|-------------|--------------|
| `VirtuosoClient` | class | Main orchestrator | SSHClient, SKILL IPC |
| `SSHClient` | class | Remote connection | SSH Tunnel, Jump Host |
| `SchematicEditor` | class | Schematic operations | VirtuosoClient |
| `LayoutEditor` | class | Layout operations | VirtuosoClient |
| `SpectreSimulator` | class | Simulation control | VirtuosoClient |
| `SKILL IPC` | mechanism | IPC protocol | ramic_bridge.il |
| `ramic_bridge.il` | script | SKILL bridge | Virtuoso environment |

### 3.2 Component Relationships

```
VirtuosoClient
    ├── SSHClient
    │   ├── SSH Tunnel
    │   └── Jump Host
    ├── SchematicEditor
    │   ├── add_instance()
    │   ├── add_wire()
    │   └── add_pin()
    ├── LayoutEditor
    │   ├── create_instance()
    │   ├── create_path()
    │   └── create_pin()
    └── SpectreSimulator
        ├── transient()
        ├── dc_ac()
        └── pss_pnoise()
```

---

## 4. VirtuosoClient API

### 4.1 Client Initialization

#### 4.1.1 Local Connection

**Constructor**:
```python
VirtuosoClient.local(
    port: int = 65432,
    timeout: int = 30
)
```

**Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `port` | int | 65432 | Local IPC port |
| `timeout` | int | 30 | Connection timeout (seconds) |

**Example**:
```python
from virtuoso_bridge import VirtuosoClient

# Local connection
client = VirtuosoClient.local(port=65432)
```

---

#### 4.1.2 Remote Connection

**Constructor**:
```python
VirtuosoClient.remote(
    host: str,
    port: int = 65432,
    ssh_host: str = None,
    ssh_port: int = 22,
    jump_host: str = None,
    jump_port: int = 22
)
```

**Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `host` | str | Required | Remote Virtuoso host |
| `port` | int | 65432 | Virtuoso IPC port |
| `ssh_host` | str | None | SSH gateway host |
| `ssh_port` | int | 22 | SSH port |
| `jump_host` | str | None | Jump host for multi-hop |
| `jump_port` | int | 22 | Jump host SSH port |

**Example**:
```python
# Remote connection with jump host
client = VirtuosoClient.remote(
    host="eda-server.local",
    ssh_host="gateway.company.com",
    jump_host="jump-proxy.company.com"
)
```

---

### 4.2 Core Methods

#### 4.2.1 execute()

**Purpose**: Execute SKILL commands

**Signature**:
```python
execute(
    commands: List[str],
    timeout: int = 60
) -> List[Any]
```

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `commands` | List[str] | SKILL command list |
| `timeout` | int | Execution timeout |

**Returns**: List of command results

**Example**:
```python
# Execute SKILL commands
result = client.execute([
    'ddGetLibList()',
    'dbOpenCellViewByType("mylib" "mycell" "schematic")'
])
```

---

#### 4.2.2 load_il()

**Purpose**: Load IL/SKILL file

**Signature**:
```python
load_il(
    path: str | Path
) -> bool
```

**Example**:
```python
# Load SKILL file
client.load_il(Path('schematic_ops.il'))
```

---

#### 4.2.3 schematic.edit()

**Purpose**: Edit schematic with context manager

**Signature**:
```python
schematic.edit(
    lib: str,
    cell: str,
    view: str = "schematic"
) -> SchematicEditor
```

**Example**:
```python
# Edit schematic
with client.schematic.edit("mylib", "amp_top") as sch:
    sch.add_instance("gpdk090", "nmos1v", (0, 0), "M1")
    sch.add_net_label_to_instance_term("M1", "D", "DRAIN")
```

---

#### 4.2.4 layout.edit()

**Purpose**: Edit layout with context manager

**Signature**:
```python
layout.edit(
    lib: str,
    cell: str,
    view: str = "layout"
) -> LayoutEditor
```

---

#### 4.2.5 close()

**Purpose**: Close connection

**Signature**:
```python
close() -> None
```

---

## 5. SchematicEditor API

### 5.1 Instance Operations

#### 5.1.1 add_instance()

**Purpose**: Add instance to schematic

**Signature**:
```python
add_instance(
    lib: str,
    cell: str,
    location: Tuple[float, float],
    name: str = None,
    orient: str = "R0"
) -> Instance
```

**Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `lib` | str | Required | Library name |
| `cell` | str | Required | Cell name |
| `location` | Tuple[float, float] | Required | (x, y) coordinates |
| `name` | str | None | Instance name (auto-generated if None) |
| `orient` | str | "R0" | Orientation (R0, R90, R180, R270, MX, MY, MXR90, MYR90) |

**Returns**: Instance object

**Example**:
```python
# Add NMOS transistor
inst = sch.add_instance(
    "gpdk090", "nmos1v", 
    (10.0, 20.0), 
    name="M1", 
    orient="R0"
)
```

---

### 5.2 Wire Operations

#### 5.2.1 add_wire()

**Purpose**: Draw wire between points

**Signature**:
```python
add_wire(
    points: List[Tuple[float, float]],
    net_name: str = None,
    layer: str = "metal1",
    width: float = 0.1
) -> Wire
```

---

#### 5.2.2 add_wire_between_instance_terms()

**Purpose**: Auto-connect two instance terminals

**Signature**:
```python
add_wire_between_instance_terms(
    inst1_name: str,
    term1: str,
    inst2_name: str,
    term2: str
) -> Wire
```

**Example**:
```python
# Connect M1 drain to M2 source
sch.add_wire_between_instance_terms("M1", "D", "M2", "S")
```

---

### 5.3 Net Operations

#### 5.3.1 add_net_label()

**Purpose**: Add net label at location

**Signature**:
```python
add_net_label(
    location: Tuple[float, float],
    net_name: str
) -> NetLabel
```

---

#### 5.3.2 add_net_label_to_instance_term()

**Purpose**: Add net label to instance terminal (auto-connects)

**Signature**:
```python
add_net_label_to_instance_term(
    inst_name: str,
    term_name: str,
    net_name: str
) -> NetLabel
```

**Example**:
```python
# Connect M1 drain to net "DRAIN"
sch.add_net_label_to_instance_term("M1", "D", "DRAIN")
```

---

### 5.4 Pin Operations

#### 5.4.1 add_pin()

**Purpose**: Add schematic pin

**Signature**:
```python
add_pin(
    name: str,
    direction: str = "inputOutput",
    location: Tuple[float, float] = None
) -> Pin
```

**Directions**: input, output, inputOutput, switch, ground, supply

---

#### 5.4.2 add_pin_to_instance_term()

**Purpose**: Add pin connected to instance terminal

**Signature**:
```python
add_pin_to_instance_term(
    inst_name: str,
    term_name: str,
    pin_name: str
) -> Pin
```

---

## 6. LayoutEditor API

### 6.1 Instance Operations

#### 6.1.1 create_instance()

**Purpose**: Create layout instance

**Signature**:
```python
create_instance(
    lib: str,
    cell: str,
    location: Tuple[float, float],
    orient: str = "R0"
) -> Instance
```

---

### 6.2 Path Operations

#### 6.2.1 create_path()

**Purpose**: Create layout path (wire)

**Signature**:
```python
create_path(
    points: List[Tuple[float, float]],
    layer: str = "metal1",
    width: float = 0.1
) -> Path
```

---

### 6.3 Pin Operations

#### 6.3.1 create_pin()

**Purpose**: Create layout pin

**Signature**:
```python
create_pin(
    name: str,
    layer: str = "metal1",
    bbox: Tuple[Tuple[float, float], Tuple[float, float]]
) -> Pin
```

---

## 7. Remote Execution

### 7.1 SSH Tunnel Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Local Host  │────▶│  Jump Host   │────▶│  EDA Server  │
│              │     │  (Optional)  │     │              │
│  AI Agent    │     │  SSH Proxy   │     │  Virtuoso    │
└──────────────┘     └──────────────┘     └──────────────┘
      │                                          │
      │                                          │
      └──────────────────────────────────────────┘
                    Persistent Tunnel
                    (Port Forwarding)
```

### 7.2 Tunnel Configuration

**Environment Variables**:
```bash
VB_REMOTE_HOST=eda-server.local
VB_REMOTE_PORT=65432
VB_SSH_HOST=gateway.company.com
VB_SSH_USER=username
VB_JUMP_HOST=jump-proxy.company.com
VB_JUMP_USER=username
```

### 7.3 Connection Methods

**Method 1: Direct SSH**:
```python
client = VirtuosoClient.remote(
    host="eda-server.local",
    ssh_host="gateway.company.com"
)
```

**Method 2: Jump Host**:
```python
client = VirtuosoClient.remote(
    host="eda-server.local",
    ssh_host="gateway.company.com",
    jump_host="jump-proxy.company.com"
)
```

### 7.4 Auto-Reconnect

**Behavior**:
- Automatic tunnel reconnection on failure
- Configurable retry count (default: 3)
- Graceful degradation on persistent failure

---

## 8. SKILL IPC Protocol

### 8.1 Protocol Specification

**Transport**: JSON-RPC over stdio  
**Version**: 1.0

### 8.2 Message Format

**Request**:
```json
{
  "jsonrpc": "2.0",
  "method": "execute",
  "id": 1,
  "params": {
    "commands": ["dbGetLibList()"]
  }
}
```

**Response**:
```json
{
  "jsonrpc": "2.0",
  "result": ["lib1", "lib2", "lib3"],
  "id": 1
}
```

### 8.3 Core Methods

| Method | Purpose | Parameters |
|--------|---------|------------|
| `execute` | Run SKILL commands | commands, timeout |
| `load_il` | Load IL file | path |
| `get_libs` | Get library list | None |
| `get_cells` | Get cell list | lib |
| `get_views` | Get view list | lib, cell |

### 8.4 ramic_bridge.il

**Location**: `/path/to/ramic_bridge.il`

**Purpose**: SKILL-side bridge script

**Key Functions**:
- `VB_Init()` - Initialize bridge
- `VB_Execute()` - Execute commands
- `VB_GetResult()` - Retrieve results
- `VB_Shutdown()` - Cleanup

---

## 9. Spectre Simulation

### 9.1 Simulation Types

| Type | Method | Description |
|------|--------|-------------|
| Transient | `transient()` | Time-domain simulation |
| DC+AC | `dc_ac()` | DC and AC analysis |
| PSS+Pnoise | `pss_pnoise()` | Periodic steady-state |
| Noise | `noise()` | Noise analysis |

### 9.2 Transient Simulation

**Signature**:
```python
transient(
    stop_time: float,
    step: float = None,
    outputs: List[str] = None
) -> SimulationResult
```

**Example**:
```python
# Run transient simulation
result = client.simulation.transient(
    stop_time=1e-6,
    outputs=["V(out)", "I(R1)"]
)
```

### 9.3 DC+AC Simulation

**Signature**:
```python
dc_ac(
    dc_vars: Dict[str, float],
    ac_freq: float,
    outputs: List[str] = None
) -> SimulationResult
```

---

### 9.4 Simulation Result

**Structure**:
```python
@dataclass
class SimulationResult:
    status: str  # "success" | "error"
    time: np.ndarray
    data: Dict[str, np.ndarray]  # {output_name: values}
    metadata: Dict[str, Any]
    error_message: str = None
```

---

## 10. Configuration System

### 10.1 Configuration File

**Location**: `~/.virtuoso-bridge/config.json`

**Schema**:
```json
{
  "version": "1.0",
  "local": {
    "port": 65432,
    "timeout": 30
  },
  "remote": {
    "default_host": "eda-server.local",
    "ssh_host": "gateway.company.com",
    "jump_host": "jump-proxy.company.com",
    "ssh_user": "${VB_SSH_USER}",
    "jump_user": "${VB_JUMP_USER}"
  },
  "skill": {
    "bridge_script": "/path/to/ramic_bridge.il",
    "timeout": 60
  },
  "simulation": {
    "spectre_path": "/opt/cadence/MMSIM151/bin/spectre",
    "timeout": 300,
    "output_dir": "/tmp/spectre_results"
  }
}
```

---

### 10.2 Environment Variables

| Variable | Purpose | Default |
|----------|---------|---------|
| `VB_REMOTE_HOST` | Remote Virtuoso host | localhost |
| `VB_REMOTE_PORT` | Virtuoso IPC port | 65432 |
| `VB_SSH_HOST` | SSH gateway | None |
| `VB_SSH_PORT` | SSH port | 22 |
| `VB_SSH_USER` | SSH username | $USER |
| `VB_JUMP_HOST` | Jump host | None |
| `VB_JUMP_PORT` | Jump host SSH port | 22 |
| `VB_JUMP_USER` | Jump host username | $USER |
| `VB_SKILL_TIMEOUT` | SKILL execution timeout | 60 |

---

## 11. Best Practices

### 11.1 Connection Management

**Recommendations**:
1. Use context managers for editing
2. Close connections explicitly
3. Handle reconnection gracefully
4. Monitor tunnel health

**Example**:
```python
# Good: Use context manager
with VirtuosoClient.local() as client:
    with client.schematic.edit("lib", "cell") as sch:
        sch.add_instance(...)

# Bad: Manual connection
client = VirtuosoClient.local()
# ... operations ...
# Forgot to close!
```

---

### 11.2 Instance Naming

**Recommendations**:
1. Use meaningful names (M1, R1, C1)
2. Follow naming conventions
3. Avoid name conflicts
4. Document naming scheme

---

### 11.3 Wire Routing

**Recommendations**:
1. Use `add_wire_between_instance_terms()` for auto-routing
2. Keep wire lengths minimal
3. Avoid crossing unrelated nets
4. Use proper layers

---

### 11.4 Net Naming

**Recommendations**:
1. Use descriptive net names (VDD, VIN, OUT)
2. Follow project conventions
3. Document net purposes
4. Use hierarchical naming

---

## 12. Security Framework

### 12.1 Security Layers

```
┌─────────────────────────────┐
│   API Authentication         │
│   (SSH Keys, Tokens)         │
├─────────────────────────────┤
│   Connection Encryption      │
│   (SSH Tunnel)              │
├─────────────────────────────┤
│   Operation Permissions      │
│   (Approval Gates)          │
├─────────────────────────────┤
│   Result Validation         │
│   (Type Checking)           │
└─────────────────────────────┘
```

---

### 12.2 SSH Key Management

**Requirements**:
- Use SSH keys (not passwords)
- Rotate keys periodically
- Use separate keys per environment
- Store keys securely

---

### 12.3 Operation Permissions

**Approval Gates**:
```json
{
  "permissions": {
    "read": ["get_libs", "get_cells", "get_views"],
    "write": ["add_instance", "add_wire", "add_pin"],
    "execute": ["transient", "dc_ac", "pss_pnoise"],
    "admin": ["delete_instance", "close"]
  }
}
```

---

## 13. Error Handling

### 13.1 Error Categories

| Category | Examples | Handling |
|----------|----------|----------|
| Connection | SSH failure, timeout | Retry 3x, reconnect |
| SKILL | Syntax error, invalid API | Log, return error |
| Virtuoso | License, session | Notify, fallback |
| Simulation | Convergence, timeout | Analyze, suggest fix |
| Remote | Tunnel drop, host down | Auto-reconnect |

---

### 13.2 Error Response

**Structure**:
```python
@dataclass
class BridgeError:
    code: str  # "CONNECTION_ERROR", "SKILL_ERROR"
    message: str
    details: Dict[str, Any]
    suggestions: List[str]
    retry_possible: bool
```

---

### 13.3 Recovery Strategies

**Connection Errors**:
```
1. Check SSH tunnel status
2. Reconnect tunnel
3. Retry operation
4. Notify if persistent
```

**SKILL Errors**:
```
1. Parse error message
2. Identify missing parameters
3. Suggest fix
4. Provide example
```

---

## 14. Integration Points

### 14.1 AI Agent Integration

**Supported Agents**:
| Agent | Integration | Features |
|-------|-------------|----------|
| Claude Code | MCP server | Schematic/Layout editing |
| Cursor | Skill file | Auto-completion |
| Copilot | VS Code extension | Inline suggestions |
| OpenCode | CLI | Remote execution |

---

### 14.2 EDA Tool Integration

**Supported Tools**:
| Tool | Integration | API Support |
|------|-------------|-------------|
| Virtuoso Schematic | ✅ Full | SchematicEditor |
| Virtuoso Layout | ✅ Full | LayoutEditor |
| ADE Maestro | ✅ Full | SpectreSimulator |
| ADE Explorer | ✅ Full | dc_ac, transient |
| Spectre | ✅ Full | All analyses |
| Ocean | ✅ Partial | execute() |

---

## 15. Development Guidelines

### 15.1 Code Style

**Python**:
- Follow PEP 8
- Use type hints
- Document with docstrings
- Prefer dataclasses

**SKILL**:
- Use meaningful function names
- Document functions
- Follow Virtuoso conventions

---

### 15.2 Testing

**Unit Tests**:
```python
def test_add_instance():
    client = VirtuosoClient.local()
    with client.schematic.edit("test_lib", "test_cell") as sch:
        inst = sch.add_instance("gpdk090", "nmos1v", (0, 0))
        assert inst.name.startswith("I")
```

---

### 15.3 Project Structure

```
virtuoso-bridge-lite/
├── src/
│   ├── virtuoso_bridge/
│   │   ├── __init__.py
│   │   ├── client.py
│   │   ├── schematic.py
│   │   ├── layout.py
│   │   ├── simulation.py
│   │   └ remote.py
│   └── skill/
│       └── ramic_bridge.il
├── tests/
├── docs/
├── examples/
└── setup.py
```

---

## 16. API Reference

### 16.1 Full API Index

| Class | Method | Signature |
|-------|--------|-----------|
| VirtuosoClient | local | `local(port=65432, timeout=30)` |
| VirtuosoClient | remote | `remote(host, ssh_host=None, jump_host=None)` |
| VirtuosoClient | execute | `execute(commands, timeout=60)` |
| VirtuosoClient | close | `close()` |
| SchematicEditor | add_instance | `add_instance(lib, cell, loc, name=None, orient="R0")` |
| SchematicEditor | add_wire | `add_wire(points, net=None, layer="metal1")` |
| SchematicEditor | add_net_label | `add_net_label(loc, net_name)` |
| SchematicEditor | add_pin | `add_pin(name, direction="inputOutput")` |
| LayoutEditor | create_instance | `create_instance(lib, cell, loc, orient="R0")` |
| LayoutEditor | create_path | `create_path(points, layer="metal1")` |
| LayoutEditor | create_pin | `create_pin(name, layer, bbox)` |
| SpectreSimulator | transient | `transient(stop_time, outputs=None)` |
| SpectreSimulator | dc_ac | `dc_ac(dc_vars, ac_freq, outputs=None)` |

---

## 17. Examples

### 17.1 Basic Connection

```python
from virtuoso_bridge import VirtuosoClient

# Local connection
client = VirtuosoClient.local(port=65432)

# Execute SKILL command
libs = client.execute(['ddGetLibList()'])
print(f"Available libraries: {libs}")

client.close()
```

---

### 17.2 Create Amplifier Schematic

```python
from virtuoso_bridge import VirtuosoClient

client = VirtuosoClient.local()

with client.schematic.edit("mylib", "amp_top") as sch:
    # Add transistors
    M1 = sch.add_instance("gpdk090", "nmos1v", (0, 0), "M1")
    M2 = sch.add_instance("gpdk090", "pmos1v", (0, 10), "M2")
    
    # Connect nets
    sch.add_net_label_to_instance_term("M1", "D", "OUT")
    sch.add_net_label_to_instance_term("M2", "D", "OUT")
    sch.add_net_label_to_instance_term("M1", "G", "VIN")
    sch.add_net_label_to_instance_term("M2", "G", "VIN")
    
    # Add pins
    sch.add_pin_to_instance_term("M1", "S", "VSS")
    sch.add_pin_to_instance_term("M2", "S", "VDD")
    sch.add_pin("VIN", "input")
    sch.add_pin("OUT", "output")

client.close()
```

---

### 17.3 Remote Execution

```python
from virtuoso_bridge import VirtuosoClient

# Remote with jump host
client = VirtuosoClient.remote(
    host="eda-server.university.edu",
    ssh_host="gateway.university.edu",
    jump_host="proxy.university.edu"
)

# Edit remote schematic
with client.schematic.edit("shared_lib", "ldo_core") as sch:
    sch.add_instance("gpdk090", "nmos1v", (5, 5), "MP")

client.close()
```

---

### 17.4 Spectre Simulation

```python
from virtuoso_bridge import VirtuosoClient

client = VirtuosoClient.local()

# Run transient simulation
result = client.simulation.transient(
    stop_time=1e-6,
    outputs=["V(out)", "I(M1.D)"]
)

if result.status == "success":
    print(f"Peak output: {max(result.data['V(out)'])}")
else:
    print(f"Error: {result.error_message}")

client.close()
```

---

## 18. Appendices

### 18.A Glossary

| Term | Definition |
|------|------------|
| VirtuosoClient | Main Python client class |
| SKILL IPC | SKILL inter-process communication |
| SSH Tunnel | Secure channel for remote execution |
| Jump Host | Intermediate SSH proxy server |
| SchematicEditor | Schematic editing context manager |
| LayoutEditor | Layout editing context manager |
| SpectreSimulator | Spectre simulation controller |

---

### 18.B Knowledge Graph Statistics

**Nodes**: 33  
**Edges**: 41  
**Communities**: 6

**Node Distribution**:
| Type | Count |
|------|-------|
| api | 10 |
| class | 5 |
| feature | 3 |
| simulation | 3 |
| mode | 2 |
| skill_api | 2 |
| tool | 2 |
| other | 8 |

---

### 18.C Related Work

| Project | Description |
|---------|-------------|
| skillbridge | Similar Python bridge, less features |
| Ocean Scripts | Cadence native scripting |
| ADE Maestro | GUI simulation environment |

---

### 18.D References

- Virtuoso Documentation: Cadence help system
- SKILL Reference: `/opt/cadence/IC617/docs/skillref.pdf`
- Spectre Reference: `/opt/cadence/MMSIM151/docs/spectreref.pdf`
- GitHub: https://github.com/Arcadia-1/virtuoso-bridge-lite

---

### 18.E Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-04-09 | Initial knowledge graph |
| 2.0 | 2026-04-16 | Detailed project specification |

---

*End of Specification Document*
