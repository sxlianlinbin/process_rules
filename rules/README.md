---
trigger: always_on
---
# Cursor AI 规则配置

本目录包含项目的 AI 助手开发规则配置文件。

## 📋 规则文件列表

### 1. ai-agent-default-behavior.mdc
**状态**: ✅ 自动应用 (`alwaysApply: true`)  
**用途**: AI 助手的默认行为规范  
**内容**: 
- 代码风格与规范（PSR-12、命名约定）
- 数据库操作规范（事务、SQL安全、查询优化）
- 权限与安全（权限检查、异常处理、数据脱敏）
- 业务逻辑规范（参数获取、返回值、配置管理）
- 并发控制（Redis锁、队列任务）
- 代码复用与维护性
- 性能优化
- 日志与监控
- 测试与调试
- 兼容性与扩展性
- 项目特定规范
- AI 助手行为准则
- 示例参考

### 2. base-rule.mdc
**状态**: ✅ 自动应用 (`alwaysApply: true`)  
**用途**: 项目基础开发规则  
**内容**:
- 第三方接口调用重试机制（3次重试，间隔1秒）
- Token 管理（统一使用 TokenService::getThirdPartyToken）
- 数据库操作基本原则（事务、参数化查询、批量处理）
- 错误处理与日志规范
- 安全规范（权限验证、数据脱敏）
- 缓存使用规范（命名、Redis锁）
- 代码规范（PSR-12、命名）
- 返回值规范（Success、分页格式）

### 3. platform-services.mdc
**状态**: 📝 按需应用 (`alwaysApply: false`)  
**用途**: 平台服务开发指南  
**适用范围**: `process/src/services/platform/` 目录  
**内容**:
- 支付服务（支付宝、校园支付）
- 短信服务（阿里云、腾讯云）
- 消息服务（微信、企业微信、钉钉）
- OAuth 登录服务
- 核心接口说明
- 使用说明和最佳实践

## 🎯 规则优先级

1. **ai-agent-default-behavior.mdc** - 最高优先级，所有场景适用
2. **base-rule.mdc** - 基础规则，强制执行
3. **platform-services.mdc** - 特定场景规则，针对平台服务开发

## 📝 规则使用指南

### 自动应用规则
标记为 `alwaysApply: true` 的规则会在所有 AI 交互中自动生效，无需手动激活。

### 场景特定规则
使用 `globs` 字段可以指定规则适用的文件路径模式，例如：
```yaml
---
globs: 
  - "process/src/services/platform/**/*.php"
alwaysApply: false
---
```

### 手动引用规则
在与 AI 对话时，可以通过以下方式引用特定规则：
- 提及规则名称："请按照 platform-services 规则创建支付服务"
- 引用规则内容："请遵循项目的事务处理规范"

## ✏️ 规则编写规范

### 文件格式
规则文件使用 Markdown Cursor (`.mdc`) 格式，支持以下元数据：

```yaml
---
description: 规则描述
globs: 
  - "path/pattern/**/*.php"
alwaysApply: true/false
---
```

### 规则内容结构
1. **标题**: 使用清晰的标题层级（H1-H4）
2. **条目**: 使用列表形式，每条规则简洁明了
3. **示例**: 提供代码示例说明正确和错误的写法
4. **强调**: 使用 **加粗** 或 `代码` 突出重点

### 规则更新流程
1. 识别新的通用规范或最佳实践
2. 确定规则应该属于哪个文件（或创建新文件）
3. 编写清晰的规则描述和示例
4. 更新本 README 文档
5. 通知团队成员规则变更

## 🔍 常见问题

### Q: 如何确认规则是否生效？
A: 查看 AI 生成的代码是否遵循规则中的规范。标记为 `alwaysApply: true` 的规则会自动应用。

### Q: 规则之间冲突怎么办？
A: 按照优先级处理，通常是更具体的规则优先级更高。如有疑问，优先遵循 `ai-agent-default-behavior.mdc`。

### Q: 可以临时禁用某个规则吗？
A: 可以将规则的 `alwaysApply` 设置为 `false`，或在与 AI 对话时明确说明不需要遵循某条规则。

### Q: 如何为特定客户添加定制规则？
A: 可以在 `process_envs/{客户}/` 目录下创建独立的规则文件，或在对话中明确说明客户特定需求。

## 📚 参考资源

- [Cursor AI 文档](https://cursor.sh/docs)
- [PSR-12 编码标准](https://www.php-fig.org/psr/psr-12/)
- [PHP 最佳实践](https://phptherightway.com/)
- [PostgreSQL 文档](https://www.postgresql.org/docs/)
- [Swoole 文档](https://wiki.swoole.com/)

## 📅 更新日志

### 2026-01-16
- ✅ 创建 `ai-agent-default-behavior.mdc` - AI 助手默认行为规则
- ✅ 更新 `base-rule.mdc` - 补充完整的基础规则
- ✅ 设置自动应用规则 (`alwaysApply: true`)
- ✅ 创建规则索引文档

---

**维护者**: 开发团队  
**最后更新**: 2026-01-16
