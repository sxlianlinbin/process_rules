---
alwaysApply: true
description: 项目基础规则 - 适用于所有开发场景
---
# 基础项目规则（Base Rule）

本规则文件为本项目的基础Cursor规则，所有开发者在开发和维护过程中需遵循。

## 1. 调用三方接口的重试机制

- 调用三方接口（如 HTTP 请求等）时，必须进行三次重试，每次重试间隔为 1 秒（1000ms）
- 若三次重试后依然失败，则需记录错误日志并抛出具有可读性的异常信息
- 使用统一的重试工具类或方法，确保重试逻辑一致

## 2. 获取 token、access_token 等具有时效性的值

- 缓存的 key 命名需遵循统一规范，建议格式：`{类名前缀}_{业务}_{唯一标识}`，如 `ApiGet_token_wechat_appId`，各部分用下划线 `_` 连接
- **获取 token 统一使用 `TokenService::getThirdPartyToken` 方法**
- Token 过期时间设置要小于实际过期时间（建议提前 5 分钟刷新）

## 3. 数据库操作基本原则

- **所有多表操作必须使用事务**
- 禁止直接拼接 SQL，必须使用参数化查询
- 批量操作使用 `array_chunk()` 分批处理，每批建议 1000 条
- Model 类必须继承框架基类，放在 `models` 命名空间

## 4. 错误处理与日志

- 用户级错误使用 `UserException`，包含友好的错误提示
- 系统级错误使用 `\Exception`
- 关键操作必须记录日志（操作人、时间、内容、结果）
- 异常处理时必须回滚事务

## 5. 安全规范

- 所有 Controller 方法必须进行权限验证
- 敏感数据（姓名、手机号、身份证等）使用 `DataSafe` 脱敏
- 防止 SQL 注入、XSS 攻击
- 输入数据必须验证和过滤

## 6. 缓存使用规范

- 缓存键命名规范：`{模块}_{功能}_{唯一标识}`
- 使用 `CacheKeyHelper` 生成统一的缓存键
- 设置合理的过期时间，避免缓存雪崩
- 关键操作使用 Redis 锁防止并发问题

## 7. 代码规范

- 遵循 PSR-12 编码标准
- 类名 PascalCase，方法名 camelCase
- Model 类以 `Model` 结尾，Controller 类以 `Controller` 结尾
- Public 方法必须有 PHPDoc 注释
- 单个方法不超过 100 行（特殊情况除外）

## 8. 返回值规范

- 成功返回：`new Success($data)`
- 空数据返回：`Success::emptySuccess()`
- 列表数据格式：`['total' => $total, 'data' => $list]`
- 分页方法添加 `#[Pagenation]` 注解