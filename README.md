# InoProShop LIMIT MCP

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![Platform](https://img.shields.io/badge/platform-Windows-lightgrey)
![InoProShop](https://img.shields.io/badge/InoProShop-V1.9.1.6-green)
![Node](https://img.shields.io/badge/node-%3E%3D18-brightgreen)

> 让 Claude AI 直接操控汇川 InoProShop，实现 PLC 程序的 AI 自动编程。
>
> Empower Claude AI to directly control Inovance InoProShop for AI-driven PLC programming automation.

---

## 简介（Chinese）

**InoProShop LIMIT MCP** 是一个基于 [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) 的服务器，专为汇川中型 PLC 编程软件 **InoProShop V1.9.1.6（SP11 内核）** 开发。

通过本服务器，Claude AI 可以：
- 创建和打开 InoProShop 项目
- 编写、读取、修改 ST 结构化文本程序
- 配置 EtherCAT 硬件拓扑和设备参数
- 管理任务调度、GVL、DUT、文件夹等工程对象
- 编译并获取真实错误信息
- 探查 SP11 IronPython 脚本 API

这使得 AI 可以像有经验的 PLC 工程师一样，端到端地完成整个自动化工程的开发。

推荐使用claude code Sonnet 4.6或更智能模型
---

## 运行要求

| 依赖 | 版本要求 |
|------|----------|
| 操作系统 | Windows（仅支持） |
| InoProShop | V1.9.1.6（SP11 内核） |
| Node.js | >= 18.0.0 |

---

## 快速配置（Claude Code）

将以下配置写入 `~/.mcp.json`（`%USERPROFILE%\\.mcp.json`）：

```json
{
  "mcpServers": {
    "codesys_local": {
      "command": "node",
      "args": [
        "path/to/bundle.min.js",
        "--codesys-path", "D:\\Inovance Control\\InoProShop\\CODESYS\\Common\\InoProShop.exe",
        "--codesys-profile", "InoProShop(V1.9.1.6)",
        "--workspace", "C:\\path\\to\\workspace"
      ]
    }
  }
}
```

> 请将 `path/to/bundle.min.js` 替换为实际的打包文件路径，`--workspace` 替换为你的工程目录。

---

## 工具列表（24 个）

### 项目管理（5 个）

| 工具 | 说明 |
|------|------|
| `open_project` | 打开已有 `.project` 工程文件 |
| `create_project` | 从标准模板创建新项目 |
| `save_project` | 保存当前项目 |
| `compile_project` | 编译主应用并返回真实错误/警告信息 |
| `get_project_structure` | 获取项目完整对象树（递归） |

### POU / 代码编辑（6 个）

| 工具 | 说明 |
|------|------|
| `create_pou` | 创建 Program / FunctionBlock / Function |
| `set_pou_code` | 写入 VAR 声明段和 ST 实现代码 |
| `get_pou_code` | 读取 POU 的声明和实现 |
| `create_method` | 在 FB 内创建 Method |
| `create_property` | 在 FB 内创建 Property（含 Get/Set） |
| `delete_pou` | 删除 POU 或任意命名对象 |

### 组织结构（5 个）

| 工具 | 说明 |
|------|------|
| `create_dut` | 创建 STRUCT / ENUM / UNION 数据类型 |
| `create_gvl` | 创建全局变量列表（GVL） |
| `create_folder` | 在应用树中创建文件夹 |
| `rename_object` | 重命名任意对象 |
| `move_object` | 将对象移动到新的父节点 |

### 硬件设备配置（4 个）

| 工具 | 说明 |
|------|------|
| `add_device` | 添加 EtherCAT 从站或其他设备节点 |
| `delete_device` | 删除设备节点 |
| `get_device_params` | 读取设备所有配置参数（含 IP、Assembly 路径等） |
| `set_device_param` | 设置指定设备参数（如 IP 地址） |

### 任务配置（3 个）

| 工具 | 说明 |
|------|------|
| `create_task` | 在任务配置中创建新任务 |
| `get_task_config` | 读取所有任务的周期、优先级和绑定 POU |
| `set_task_config` | 修改任务的周期（μs）和优先级 |

### 调试工具（1 个）

| 工具 | 说明 |
|------|------|
| `probe_api` | 探查 SP11 IronPython API（支持 dir / children / connectors / custom 模式） |

---

## 使用建议

### 推荐工作流程

1. **先用 `get_project_structure` 了解项目全貌**，再开始修改，避免重复创建或路径错误。
2. **写代码前先用 `get_pou_code` 读取现有内容**，在原有基础上修改，不要直接覆盖。
3. **每完成一个模块立即 `save_project`**，InoProShop 崩溃不会自动恢复。
4. **用 `compile_project` 验证每次改动**，错误信息直接定位到行号和对象名。

### 硬件项目注意事项

- `create_project` 创建的是软仿真项目（PLCWinNT），**无法添加 EtherCAT 设备**。
- 需要 EtherCAT/伺服设备时，请以含硬件控制器的项目为基础（`Device [type=4102]` 节点下才有 EtherCAT 插槽）。
- 添加设备前先查 `devicecache.xml` 确认 type / id / version，或用 `probe_api` 探查已有节点结构。

### ST 编程建议

- 上升沿触发指令（MC_MoveAbsolute 等）的 Execute 必须用 R_TRIG 产生脉冲，不能保持 TRUE。
- Acceleration / Deceleration 参数必须 > 0，否则编译通过但运行报错。
- 轴对象（AXIS_REF_SM3）在功能块间传递必须用 `VAR_IN_OUT`，不能用 `VAR_INPUT`。
- MC_Stop 在 SP11 中没有 Done/Error 输出参数，只有 Busy 和 ErrorID。

### probe_api 调试技巧

当工具行为不符合预期时，用 `probe_api` 的 `custom` 模式直接执行 IronPython 脚本探查内部状态，例如读取节点属性、遍历子对象等，有助于快速定位问题。

---

## 已知不支持项

以下功能受限于 SP11 脚本 API，暂无法通过 MCP 实现：

- 向任务添加 / 删除 POU 调用
- IO 通道变量映射（Mapping）
- EtherCAT PDO 映射配置

---

## 适用设备

汇川中型 PLC AM400 / AM500 / AM600 系列，配合 InoProShop V1.9.1.6。

---

---

## Introduction (English)

**InoProShop LIMIT MCP** is a [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server built specifically for **Inovance InoProShop V1.9.1.6 (SP11 kernel)** — the PLC programming IDE for Inovance mid-range PLCs.

With this server, Claude AI can:
- Create and open InoProShop projects
- Write, read, and modify Structured Text (ST) programs
- Configure EtherCAT hardware topology and device parameters
- Manage tasks, GVLs, DUTs, folders, and other project objects
- Compile projects and retrieve real error messages
- Probe the SP11 IronPython scripting API

This enables AI to complete end-to-end PLC automation projects like an experienced engineer.

---

## Requirements

| Dependency | Version |
|------------|---------|
| OS | Windows only |
| InoProShop | V1.9.1.6 (SP11 kernel) |
| Node.js | >= 18.0.0 |

---

## Quick Setup (Claude Code)

Add the following to `~/.mcp.json` (`%USERPROFILE%\\.mcp.json`):

```json
{
  "mcpServers": {
    "codesys_local": {
      "command": "node",
      "args": [
        "path/to/bundle.min.js",
        "--codesys-path", "D:\\Inovance Control\\InoProShop\\CODESYS\\Common\\InoProShop.exe",
        "--codesys-profile", "InoProShop(V1.9.1.6)",
        "--workspace", "C:\\path\\to\\workspace"
      ]
    }
  }
}
```

> Replace `path/to/bundle.min.js` with the actual path to the bundled JS file, and `--workspace` with your project directory.

---

## Tools (24 total)

### Project Management (5)

| Tool | Description |
|------|-------------|
| `open_project` | Open an existing `.project` file |
| `create_project` | Create a new project from the standard template |
| `save_project` | Save the current project |
| `compile_project` | Build the application and return real errors/warnings |
| `get_project_structure` | Get the full recursive object tree of the project |

### POU / Code Editing (6)

| Tool | Description |
|------|-------------|
| `create_pou` | Create a Program, FunctionBlock, or Function |
| `set_pou_code` | Write VAR declarations and ST implementation code |
| `get_pou_code` | Read a POU's declaration and implementation |
| `create_method` | Create a Method inside a Function Block |
| `create_property` | Create a Property (with Get/Set) inside a Function Block |
| `delete_pou` | Delete a POU or any named object |

### Organization (5)

| Tool | Description |
|------|-------------|
| `create_dut` | Create a STRUCT, ENUM, or UNION data type |
| `create_gvl` | Create a Global Variable List (GVL) |
| `create_folder` | Create a folder in the application tree |
| `rename_object` | Rename any named object |
| `move_object` | Move an object to a new parent node |

### Hardware / Device Configuration (4)

| Tool | Description |
|------|-------------|
| `add_device` | Add an EtherCAT slave or other device node |
| `delete_device` | Delete a device node |
| `get_device_params` | Read all host parameters of a device (IP, Assembly paths, etc.) |
| `set_device_param` | Set a specific device parameter (e.g., IP address) |

### Task Configuration (3)

| Tool | Description |
|------|-------------|
| `create_task` | Create a new task in Task Configuration |
| `get_task_config` | Read all tasks: interval, priority, and assigned POUs |
| `set_task_config` | Modify a task's cycle interval (μs) and priority |

### Debug / Inspection (1)

| Tool | Description |
|------|-------------|
| `probe_api` | Probe the SP11 IronPython API (modes: dir / children / connectors / custom) |

---

## Usage Tips

### Recommended Workflow

1. **Start with `get_project_structure`** to understand the project layout before making changes.
2. **Read before writing** — use `get_pou_code` to see existing code before overwriting it.
3. **Save frequently** with `save_project` — InoProShop does not auto-recover on crash.
4. **Compile after every change** with `compile_project` to get line-accurate error messages.

### Hardware Project Notes

- `create_project` generates a software simulation project (PLCWinNT) — **EtherCAT devices cannot be added** to it.
- For EtherCAT / servo projects, start from an existing AM600 hardware project (a `Device [type=4102]` node is required for EtherCAT slots).
- Check `devicecache.xml` for the correct type / id / version before calling `add_device`, or use `probe_api` to inspect existing node structure.

### ST Programming Tips

- Execute-triggered FBs (e.g. `MC_MoveAbsolute`) require a rising edge pulse via `R_TRIG` — holding Execute = TRUE only fires once.
- `Acceleration` and `Deceleration` must always be > 0 or a runtime error will occur.
- Axis objects (`AXIS_REF_SM3`) must be passed via `VAR_IN_OUT` — using `VAR_INPUT` copies the struct and breaks communication.
- `MC_Stop` in SP11 has no `Done` or `Error` output — only `Busy` and `ErrorID`.

### probe_api Debug Tips

When tool behavior is unexpected, use `probe_api` in `custom` mode to execute IronPython scripts directly against the open project — useful for reading node properties, enumerating children, or verifying internal state.

---

## Known Limitations

The following features are not available due to SP11 scripting API restrictions:

- Adding / removing POU calls from tasks
- IO channel variable mapping
- EtherCAT PDO mapping configuration

---

## Target Hardware

Inovance AM400 / AM500 / AM600 series mid-range PLCs with InoProShop V1.9.1.6.

---

## License

LIMINGTONG
