# InoProShop LIMIT MCP

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

---

## 运行要求

| 依赖 | 版本要求 |
|------|----------|
| 操作系统 | Windows（仅支持） |
| InoProShop | V1.9.1.6（SP11 内核） |
| Node.js | >= 18.0.0 |

---

## 快速配置（Claude Code）

将以下配置写入 `~/.mcp.json`（`%USERPROFILE%\.mcp.json`）：

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

Add the following to `~/.mcp.json` (`%USERPROFILE%\.mcp.json`):

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

MIT
