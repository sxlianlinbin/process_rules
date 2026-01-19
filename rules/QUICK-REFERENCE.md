---
trigger: always_on
---
# 🚀 开发规则快速参考

这是一个精简版的规则速查表，用于日常开发快速查阅。

## ⚡ 核心规则（必须遵守）

### 1️⃣ 数据库操作
```php
// ✅ 正确：使用事务
$transaction = db()->beginTransaction();
try {
    Model::insert([...]);
    Model::update([...]);
    $transaction->commit();
} catch (\Exception $e) {
    $transaction->rollBack();
    throw new UserException('操作失败：' . $e->getMessage());
}

// ❌ 错误：直接拼接 SQL
$sql = "WHERE id = {$id}"; // 危险！

// ✅ 正确：参数化查询
$sql = "WHERE id = :id";
db()->createCommand($sql, [':id' => $id])->queryAll();
```

### 2️⃣ 权限检查
```php
// ✅ 每个 Controller 方法必须检查权限
public function updateUser()
{
    $this->checkPermission('user.update'); // 必须
    // 业务逻辑...
}
```

### 3️⃣ 异常处理
```php
// ✅ 用户友好的错误信息
throw new UserException('操作失败，请稍后重试');

// ❌ 不要暴露技术细节给用户
throw new UserException($e->getMessage()); // 可能暴露敏感信息
```

### 4️⃣ 参数获取
```php
// ✅ 使用框架方法
$id = $this->getInt('id', 0);        // GET 参数
$name = $this->postStr('name');      // POST 参数
$data = $this->post();               // POST 全部数据

// ✅ 必填参数验证
if (!$name) throw new UserException('名称不能为空');
```

### 5️⃣ 返回值规范
```php
// ✅ 成功返回
return new Success($data);

// ✅ 空数据返回
return Success::emptySuccess();

// ✅ 列表数据格式
return new Success([
    'total' => $total,
    'data' => $list
]);

// ✅ 分页方法添加注解
#[Pagenation]
public function getUserList($offset, $limit) { }
```

## 🔐 安全规则

### 数据脱敏
```php
// ✅ 敏感数据脱敏
$showPlain = DataSafe::isShowPlaintext(user()->id);
!$showPlain && $data = DataSafe::formatResult($data, [
    'name' => 'users__name',
    'mobile' => 'users__mobile'
]);
```

### Redis 锁（防并发）
```php
// ✅ 关键操作使用锁
$redisKey = CacheKeyHelper::user_sync_mode($syncMode, $id);
if (redis()->get($redisKey)) {
    throw new UserException('操作正在进行中，请稍后再试');
}
redis()->setex($redisKey, 3600, 1);
```

## 📦 代码复用

### 批量操作
```php
// ✅ 大量数据分批处理
$chunks = array_chunk($numbers, 1000);
foreach ($chunks as $chunk) {
    UsersModel::update(['status' => 1], ['number' => $chunk]);
}
```

### 队列任务
```php
// ✅ 耗时操作使用队列
$task = new \services\task\UserSync([
    'triggerMode' => UserSyncLogModel::TRIGGER_MODE_MANUAL,
    'syncMode' => $syncMode,
]);
queue()->push(Queue::TOPIC_COMMON, $task);
```

## 🎨 命名规范

| 类型 | 规范 | 示例 |
|-----|------|-----|
| 类名 | PascalCase | `UserSyncModel`, `SystemController` |
| 方法名 | camelCase | `getUserList()`, `setConfig()` |
| 变量名 | camelCase | `$userId`, `$syncMode` |
| 常量 | UPPER_SNAKE_CASE | `STATUS_ACTIVE`, `TYPE_DEPART` |
| Model | 以 Model 结尾 | `UsersModel`, `TeamModel` |
| Controller | 以 Controller 结尾 | `UserController`, `SystemController` |

## 📂 目录结构

```
process/src/
├── http/           # Controller 层
│   ├── system/     # 系统模块
│   └── api/        # API 接口
├── models/         # Model 层（以 Model 结尾）
├── services/       # Service 层
│   ├── user/       # 用户服务
│   └── platform/   # 平台服务
└── helpers/        # 工具类（以 Helper 结尾）
```

## 🔧 第三方接口

### 重试机制
```php
// ✅ 三次重试，间隔 1 秒
for ($i = 0; $i < 3; $i++) {
    try {
        $result = $this->callApi();
        break;
    } catch (\Exception $e) {
        if ($i === 2) throw $e; // 最后一次失败则抛出
        sleep(1);
    }
}
```

### Token 获取
```php
// ✅ 统一使用 TokenService
$token = TokenService::getThirdPartyToken($platform, $config);

// ✅ 缓存键命名规范
$cacheKey = CacheKeyHelper::api_token($platform, $appId);
```

## 📊 查询优化

```php
// ✅ 单条查询
$user = UsersModel::findOne(['id' => $id]);

// ✅ 单字段查询
$name = UsersModel::scalar(['id' => $id], 'name');

// ✅ 列表查询（分页）
$list = UsersModel::lists($where, $offset, $limit, ['id desc']);

// ✅ 全部查询（谨慎使用）
$all = UsersModel::listNoPage(['status' => 1]);

// ⚠️ 避免 N+1 查询
// 使用 JOIN 或预加载
```

## 📝 注释规范

```php
/**
 * 用户数据同步
 * @param int $offset 偏移量
 * @param int $limit 每页数量
 * @return Success
 * @throws UserException
 */
#[Pagenation]
public function getUserSyncLog($offset, $limit)
{
    // 获取同步配置
    $config = UserSyncConfigModel::findOne(['id' => $id]);
    
    // 复杂逻辑添加注释
    // 注意：这里需要过滤已删除的记录
    $where[] = ['=', 'is_del', 0];
    
    return new Success($data);
}
```

## ⚠️ 常见错误

### ❌ 错误示例
```php
// ❌ 没有事务
Model1::insert([...]);
Model2::insert([...]);

// ❌ SQL 拼接
$sql = "WHERE name = '{$name}'";

// ❌ 没有权限检查
public function deleteUser() {
    UsersModel::delete(['id' => $id]);
}

// ❌ 错误信息不友好
throw new UserException($e->getMessage());

// ❌ 在循环中查询
foreach ($users as $user) {
    $depart = DepartmentModel::findOne(['id' => $user['depart_id']]);
}
```

### ✅ 正确示例
```php
// ✅ 使用事务
$transaction = db()->beginTransaction();
try {
    Model1::insert([...]);
    Model2::insert([...]);
    $transaction->commit();
} catch (\Exception $e) {
    $transaction->rollBack();
    throw new UserException('操作失败');
}

// ✅ 参数化查询
$where = ['name' => $name];

// ✅ 权限检查
public function deleteUser() {
    $this->checkPermission('user.delete');
    UsersModel::delete(['id' => $id]);
}

// ✅ 友好错误信息
throw new UserException('删除失败，请稍后重试');

// ✅ 预查询
$departIds = array_column($users, 'depart_id');
$departs = DepartmentModel::listNoPage(['id' => $departIds]);
$departMap = array_column($departs, null, 'id');
```

## 🎯 性能优化检查清单

- [ ] 多表操作是否使用事务？
- [ ] 是否有 SQL 拼接？
- [ ] 批量操作是否分批处理？
- [ ] 是否有 N+1 查询？
- [ ] 频繁访问的数据是否缓存？
- [ ] 关键操作是否有 Redis 锁？
- [ ] 耗时操作是否使用队列？

## 🔍 代码审查检查清单

- [ ] 是否有权限检查？
- [ ] 异常处理是否完整？
- [ ] 敏感数据是否脱敏？
- [ ] 返回值格式是否规范？
- [ ] 命名是否符合规范？
- [ ] 是否有必要的注释？
- [ ] 是否有代码重复？

---

**提示**: 详细规则请查看 `.cursor/rules/` 目录下的完整文档。

**快速访问**:
- [AI 助手默认行为规则](ai-agent-default-behavior.mdc)
- [基础项目规则](base-rule.mdc)
- [平台服务指南](platform-services.mdc)
- [规则索引](README.md)
