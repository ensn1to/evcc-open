# evcc 项目 MCP (Model Context Protocol) 功能分析报告

## 概述

evcc 项目集成了 MCP (Model Context Protocol) 功能，为 AI 助手（如 Claude Desktop）提供了直接访问和控制 evcc 能源管理系统的能力。MCP 是一个开放标准，允许 AI 助手安全地与外部系统交互。

## 架构组件

### 1. 核心文件结构

```
server/mcp/
├── mcp.go          # 主要的 MCP 处理器实现
├── tools.go        # 自定义工具定义
├── prompt.go       # 智能提示处理器
├── prompt.tpl      # 充电计划提示模板
├── openapi.json    # API 规范文件（嵌入式）
└── openapi.md      # API 文档
```

### 2. 主要组件分析

#### 2.1 MCP 处理器 (`mcp.go`)

**功能职责：**
- 创建和配置 MCP 服务器实例
- 加载和解析 OpenAPI 规范
- 注册 API 工具和自定义工具
- 处理 HTTP 请求代理

**关键实现：**
```go
func NewHandler(host http.Handler, baseUrl, basePath string) (http.Handler, error)
```

**工作流程：**
1. 解析嵌入的 OpenAPI 规范文件
2. 创建 MCP 服务器实例（名称："evcc"，版本：util.Version）
3. 自动注册 OpenAPI 工具（基于标签过滤）
4. 注册自定义工具和提示
5. 返回可流式处理的 HTTP 处理器

#### 2.2 工具注册

**API 工具自动生成：**
- 基于 OpenAPI 规范自动生成 MCP 工具
- 支持的 API 标签：
  - `general` - 通用功能
  - `tariffs` - 电价管理
  - `loadpoints` - 充电点控制
  - `vehicles` - 车辆管理
  - `battery` - 电池控制

**自定义工具：**
- `docs` 工具：提供文档链接（https://docs.evcc.io）

#### 2.3 智能提示系统

**充电计划提示：**
- 名称：`create-charge-plan`
- 参数：`loadpoint`（充电点）、`vehicle`（车辆）
- 功能：生成优化的充电计划

**提示模板特性：**
- 考虑家用电池状态和可控性
- 整合电网电价和反馈电价
- 包含太阳能发电预测
- 优化总体成本
- 提供计划说明和成本分析

### 3. 系统集成

#### 3.1 启用方式

**命令行配置：**
```bash
evcc --mcp
```

**代码实现：**
```go
// cmd/root.go
if viper.GetBool("mcp") {
    const path = "/mcp"
    local := conf.Network.URI()
    router := httpd.Router()
    
    var handler http.Handler
    if handler, err = mcp.NewHandler(router, local, path); err == nil {
        router.PathPrefix(path).Handler(handler)
    }
}
```

#### 3.2 网络配置

- **默认端口：** 7070
- **MCP 路径：** `/mcp`
- **API 基础路径：** `/api`
- **完整访问地址：** `http://localhost:7070/mcp`

#### 3.3 请求处理机制

**代理模式：**
- MCP 工具调用通过内部代理转发到主 HTTP 路由器
- 使用 `httptest.NewRecorder()` 模拟 HTTP 请求
- 确保安全访问受保护的 API 端点

## 功能特性

### 1. 自动化 API 访问

**支持的操作类别：**

#### 认证管理
- 管理员登录/登出
- 密码更改
- 认证状态查询

#### 电池控制
- 禁用外部电池控制
- 设置电池放电控制
- 电网充电限制管理

#### 充电点管理
- 充电模式控制
- 电流和功率设置
- 充电计划管理

#### 车辆管理
- 车辆状态查询
- SOC（电量状态）管理
- 充电参数配置

#### 电价管理
- 动态电价查询
- 成本优化计算

### 2. 智能决策支持

**充电计划优化：**
- 综合考虑电价、太阳能预测、电池状态
- 提供成本最优的充电策略
- 支持多车辆和多充电点场景

**决策因素：**
- 家用电池可用性和可控性
- 电网电价和反馈电价
- 太阳能发电预测
- 车辆充电需求
- 总体成本优化

### 3. 安全性设计

**访问控制：**
- 通过内部代理确保安全访问
- 保持原有的认证和授权机制
- 不直接暴露内部 API

**数据保护：**
- 请求和响应通过内部处理
- 维护现有的安全策略

## 技术实现细节

### 1. 依赖库

```go
// 主要依赖
"github.com/modelcontextprotocol/go-sdk/mcp"           // MCP Go SDK
"github.com/evcc-io/openapi-mcp"                      // OpenAPI 到 MCP 转换
"github.com/getkin/kin-openapi/openapi3"              // OpenAPI 解析
"github.com/Masterminds/sprig/v3"                     // 模板函数
```

### 2. 数据流

```
MCP 客户端 → HTTP 请求 → MCP 处理器 → 工具调用 → API 代理 → evcc 核心 API → 响应返回
```

### 3. 错误处理

- OpenAPI 规范解析错误处理
- 工具调用异常捕获
- 模板渲染错误处理
- HTTP 代理错误传播

## 使用场景

### 1. AI 助手集成

**Claude Desktop 集成：**
- 直接通过自然语言控制 evcc 系统
- 智能充电计划生成
- 实时状态查询和分析

### 2. 自动化运维

**系统监控：**
- 自动化状态检查
- 异常情况处理
- 性能优化建议

### 3. 智能决策

**能源管理优化：**
- 基于实时数据的充电策略
- 成本效益分析
- 可再生能源利用最大化

## 配置和部署

### 1. 启用 MCP

```bash
# 启动时启用 MCP
evcc --mcp

# 或在配置文件中设置
mcp: true
```

### 2. 客户端配置

**Claude Desktop 配置示例：**
```json
{
  "mcpServers": {
    "evcc": {
      "command": "curl",
      "args": ["-X", "POST", "http://localhost:7070/mcp"]
    }
  }
}
```

### 3. 网络要求

- 确保端口 7070 可访问
- 配置适当的防火墙规则
- 考虑 HTTPS 配置（生产环境）

## 优势和价值

### 1. 技术优势

- **标准化接口：** 基于 MCP 开放标准
- **自动化生成：** 基于 OpenAPI 自动生成工具
- **类型安全：** 完整的类型定义和验证
- **可扩展性：** 支持自定义工具和提示

### 2. 业务价值

- **智能化运维：** AI 驱动的系统管理
- **成本优化：** 智能充电计划降低能源成本
- **用户体验：** 自然语言交互界面
- **决策支持：** 基于数据的智能建议

### 3. 生态系统集成

- **AI 助手生态：** 与主流 AI 助手无缝集成
- **开发者友好：** 标准化的 API 接口
- **社区支持：** 基于开源标准

## 总结

evcc 的 MCP 功能实现了一个完整的 AI 助手集成解决方案，通过标准化的 MCP 协议为 AI 助手提供了安全、高效的系统访问能力。该实现不仅支持基本的 API 操作，还提供了智能决策支持，特别是在充电计划优化方面，体现了 AI 技术在能源管理领域的实际应用价值。

这个功能的设计充分考虑了安全性、可扩展性和易用性，为 evcc 系统的智能化升级提供了坚实的技术基础。