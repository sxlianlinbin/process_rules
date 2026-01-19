---
alwaysApply: true
description: AI智能体默认行为规则 - 适用于所有代码生成和修改操作
---

# AI 智能体默认行为规则

本规则定义了AI助手在协助开发时应遵循的默认行为和最佳实践。

## 1. 代码风格与规范

### 1.1 PHP 编码规范
- 遵循 PSR-12 编码标准
- 类名使用 PascalCase（首字母大写驼峰），如 `UserSyncModel`
- 方法名和变量名使用 camelCase（小驼峰），如 `getUserList()`
- 常量使用全大写下划线分隔，如 `STATUS_ACTIVE`
- 所有 Model 类必须以 `Model` 结尾，如 `UsersModel`
- 所有 Controller 类必须以 `Controller` 结尾，如 `SystemController`

### 1.2 命名空间规范
- Controller 放在 `http\` 命名空间下
- Model 放在 `models\` 命名空间下
- Service 放在 `services\` 命名空间下
- Helper 放在 `helpers\` 命名空间下

### 1.3 注释规范
- 所有 public 方法必须有 PHPDoc 注释
- 复杂业务逻辑必须添加行内注释说明
- 注释应简洁明了，避免冗余
- 类注释说明类的主要职责

## 2. 数据库操作规范

### 2.1 事务处理（强制）
- **所有涉及多个表操作的方法必须使用事务**
- 事务必须包含完整的 try-catch-rollback 结构
- 示例：
```php
$transaction = db()->beginTransaction();
try {
    // 数据库操作
    Model::insert([...]);
    Model::update([...]);
    $transaction->commit();
} catch (\Exception $e) {
    $transaction->rollBack();
    throw new UserException('操作失败：' . $e->getMessage());
}
```

### 2.2 SQL 安全
- **严禁直接拼接 SQL 语句**，必须使用参数化查询
- 使用 Model 的查询构造器或 `db()->createCommand()` 的参数绑定
- 错误示例：`WHERE id = {$id}` ❌
- 正确示例：`WHERE id = :id` 并绑定参数 `:id => $id` ✅

### 2.3 查询优化
- 避免 N+1 查询问题
- 使用 `listNoPage()` 获取全部数据时要谨慎，考虑数据量
- 列表查询优先使用 `lists($where, $offset, $limit, $order)`
- 单条查询使用 `findOne($where)`
- 获取单个字段值使用 `scalar($where, $field)`

## 3. 权限与安全

### 3.1 权限检查（强制）
- **所有 Controller 方法必须进行权限验证**
- 使用统一的权限检查方法，如 `checkPermission($type)`
- 涉及用户数据的操作必须验证当前用户身份

### 3.2 异常处理
- 用户相关错误使用 `UserException`
- 系统错误可以使用通用 `\Exception`
- 异常信息必须对用户友好，避免暴露敏感信息
- 错误信息中的技术术语要转换为业务术语

### 3.3 数据安全
- 敏感数据（姓名、学工号、手机号等）使用 `DataSafe` 进行脱敏
- 使用 `DataSafe::isShowPlaintext()` 检查是否展示明文
- 日志记录避免输出敏感信息

## 4. 业务逻辑规范

### 4.1 参数获取
- GET 参数使用 `$this->getInt()` / `$this->getStr()`
- POST 参数使用 `$this->post()` / `$this->postInt()` / `$this->postStr()`
- 必填参数必须验证，为空时抛出 `UserException`

### 4.2 返回值规范
- 成功响应使用 `new Success($data)`
- 无数据返回使用 `Success::emptySuccess()`
- 列表数据返回格式：`['total' => $total, 'data' => $list]`
- 分页方法必须添加 `#[Pagenation]` 注解

### 4.3 配置管理
- 配置信息存储为 JSON 格式
- 配置修改必须记录日志（如 ConfigLogModel）
- 重要配置变更需要事务保护

## 5. 并发控制

### 5.1 Redis 锁机制
- 对于可能并发执行的关键操作，必须使用 Redis 锁
- 使用 `CacheKeyHelper` 生成统一的缓存键
- 示例：
```php
$redisKey = CacheKeyHelper::user_sync_mode($syncMode, $id);
if (redis()->get($redisKey)) {
    throw new UserException('操作正在进行中，请稍后再试');
}
redis()->setex($redisKey, 3600, 1);
```

### 5.2 队列任务
- 耗时操作使用队列异步处理
- 队列任务推送使用 `queue()->push(Queue::TOPIC_COMMON, $task)`
- 任务类必须继承相应的基类

## 6. 代码复用与维护性

### 6.1 避免代码重复
- 相似逻辑超过 3 次重复时，必须提取为方法
- 通用功能封装到 Service 层或 Helper
- 使用继承和 Trait 复用代码

### 6.2 方法职责单一
- 单个方法不应超过 100 行（特殊情况除外）
- 方法应只做一件事，职责清晰
- 复杂业务逻辑拆分为多个私有方法

### 6.3 配置驱动
- 重复的配置信息提取为常量或配置数组
- 使用配置数组代替 switch-case 或大量 if-else
- 错误信息配置化（参考 `$errorMessages` 数组）

## 7. 性能优化

### 7.1 数据库性能
- 批量操作使用 `array_chunk()` 分批处理（建议每批 1000 条）
- 避免在循环中执行数据库查询
- 使用索引字段作为查询条件

### 7.2 缓存策略
- 频繁访问的数据使用缓存
- 缓存键命名规范：`{模块}_{功能}_{唯一标识}`
- 缓存时间根据业务特点设置合理的过期时间

## 8. 日志与监控

### 8.1 日志记录
- 关键操作必须记录日志
- 日志包含：操作人、操作时间、操作内容、操作结果
- 使用专门的 Log Model 记录业务日志

### 8.2 操作追踪
- 配置修改记录历史版本
- 数据同步记录详细日志（成功/失败/忽略）
- 异常情况记录完整的错误信息和堆栈

## 9. 测试与调试

### 9.1 接口测试
- 提供测试连通性的方法（如 `runTest()`）
- 测试方法不应修改生产数据
- 返回清晰的测试结果

### 9.2 数据回滚
- 提供数据恢复或回滚机制
- 删除操作优先使用软删除（状态标记）
- 关键数据删除前备份

## 10. 兼容性与扩展性

### 10.1 向后兼容
- 修改已有接口时保持向后兼容
- 新增可选参数而非修改必填参数
- 废弃功能使用 @deprecated 标记

### 10.2 可扩展设计
- 使用策略模式处理多种类型
- 配置驱动的设计便于扩展
- 预留扩展点和回调钩子

## 11. 项目特定规范

### 11.1 目录结构
- Controller 放在 `process/src/http/` 目录
- Model 放在 `process/src/models/` 目录
- Service 放在 `process/src/services/` 目录
- 环境特定代码放在 `process_envs/{客户}/{环境}/` 目录

### 11.2 多租户支持
- 注意客户特定代码的隔离
- 使用 `process_envs` 目录管理不同客户的定制代码
- 通用功能放在 `process/src` 目录

### 11.3 数据库兼容
- 支持 PostgreSQL 和 OpenGauss
- JSON 字段查询使用 `->>` 操作符
- 注意 PostgreSQL 特有的语法和函数

## 12. AI 助手行为准则

### 12.1 代码生成原则
- **优先参考项目现有代码风格和模式**
- 生成的代码应立即可运行，无需手动修改
- 包含所有必需的 import/use 语句
- 变量和方法命名要有意义，见名知意

### 12.2 代码修改原则
- 理解整体逻辑后再修改，避免破坏已有功能
- 保持现有代码风格一致
- 修改时考虑影响范围，必要时修改相关代码
- 重构代码时保持功能不变

### 12.3 问题解决方式
- 遇到不确定的情况，先查看项目中的类似实现
- 优先使用项目已有的工具类和方法
- 避免引入新的依赖，除非确有必要
- 提供清晰的注释说明复杂逻辑

### 12.4 响应用户请求
- 准确理解用户需求，必要时澄清
- 提供完整的解决方案，而非片段代码
- 解释关键决策和设计选择
- 指出潜在问题和注意事项

## 13. 示例参考

在实现功能时，可以参考以下优秀示例：
- 事务处理：`UserSync::setConfig()`
- 权限检查：`UserSync::checkPermission()`
- 分页查询：`UserSync::getSyncLog()`
- 配置管理：`UserSync::setConfig()` 的配置结构
- 异常处理：`UserSync::deleteNoValidData()`
- Redis 锁：`UserSync::syncRun()` 的并发控制
- 队列任务：`UserSync::syncRun()` 的队列推送

---

**重要提示**：以上规则是强制性的默认行为，AI 助手在所有代码生成和修改操作中都应严格遵守。如有特殊情况需要偏离规则，必须明确说明原因。
