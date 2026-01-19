---
trigger: always_on
---
# 项目开发综合规则手册

本手册整合了项目的所有开发规则，涵盖了从代码规范到架构设计的各个方面，旨在为开发人员提供一套完整、清晰的开发指导。

## 1. 代码规范

### 1.1 命名规范
- 类名使用帕斯卡命名法（PascalCase），如 `UserController`, `MapConfigModel`，首字母大写驼峰命名
- 方法名和变量名使用驼峰命名法（camelCase），如 `getUserList()`, `userName`
- 常量使用全大写下划线分隔，如 `STATUS_ACTIVE`, `TYPE_LOGIN_CONFIG`
- 变量名使用驼峰命名法，具有描述性，避免单字母变量（循环变量除外）
- 所有 Model 类必须以 `Model` 结尾，如 `UsersModel`, `UserSyncModel`
- 缓存键命名规范：`{模块}_{功能}_{唯一标识}`，如 `user_sync_mode_123`

### 1.2 代码格式
- 遵循 PSR-12 编码标准
- 使用4个空格作为缩进，不允许使用Tab
- 每行代码不超过120个字符
- 类的左大括号 `{` 必须放在方法声明的下一行
- 控制结构关键字(if, for, while等)后必须有一个空格
- 控制结构的左大括号 `{` 必须跟关键字在同一行

### 1.3 注释规范
- 类必须包含PHPDoc注释，描述类的功能和作者信息
- 所有 public 方法必须有 PHPDoc 注释，包含参数、返回值和异常说明
- 复杂业务逻辑必须添加行内注释说明
- 注释应简洁明了，避免冗余
- 类注释说明类的主要职责
- 复杂逻辑块前添加行内注释解释其作用

### 1.4 命名空间规范
- Controller 放在 `http\` 命名空间下
- Model 放在 `models\` 命名空间下
- Service 放在 `services\` 命名空间下
- Helper 放在 `helpers\` 命名空间下

#### 核心代码命名空间规范
核心代码 process/src/ 目录下的类文件命名空间从 src 后面的目录开始：
```
process/src/
├── word/
│   ├── Axx.php        # 命名空间应为: word
│   └── Bxx.php        # 命名空间应为: word
├── excel/
│   ├── Cxx.php        # 命名空间应为: excel
│   └── processors/
│       └── Dxx.php    # 命名空间应为: excel\processors
└── pdf/
    ├── Exx.php        # 命名空间应为: pdf
    └── utils/
        └── Fxx.php    # 命名空间应为: pdf\utils
```

#### 定制化代码命名空间规范
定制化代码 process_envs/cust/env 目录下的类文件命名空间：
1. env 目录下的子目录固定为 product 和 test
2. 命名空间从 env 后面的目录开始
3. 命名空间前面固定增加一层 envs

```
process_envs/cust/env/
├── product/                    # 生产环境 → 命名空间: envs\product
│   ├── services/
│   │   ├── ProductService.php    # 命名空间: envs\product\services
│   │   └── PaymentService.php    # 命名空间: envs\product\services
│   ├── processors/
│   │   └── ProductProcessor.php  # 命名空间: envs\product\processors
│   └── ProductConfig.php       # 命名空间: envs\product
└── test/                       # 测试环境 → 命名空间: envs\test
    ├── services/
    │   ├── TestProductService.php # 命名空间: envs\test\services
    │   └── MockPaymentService.php # 命名空间: envs\test\services
    ├── processors/
    │   └── TestProcessor.php    # 命名空间: envs\test\processors
    └── TestConfig.php          # 命名空间: envs\test
```

## 2. 架构规范

### 2.1 MVC架构
- 控制器(Controller)负责处理HTTP请求和响应，不应包含业务逻辑
- 模型(Model)负责数据操作和业务规则，继承自基础Model类
- 服务(Service)层处理复杂业务逻辑，保持控制器轻量化

### 2.2 类职责分离
- 每个类应该只负责一个明确的功能领域
- 控制器方法应尽量简短，复杂逻辑委托给服务类
- 模型类专注于数据操作和验证逻辑
- 避免在控制器中直接进行数据库操作

## 3. 数据库操作规范

### 3.1 事务处理（强制）
- **所有涉及多个表操作的方法必须使用事务**
- **所有多表操作必须使用事务**
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

### 3.2 SQL 安全
- **严禁直接拼接 SQL 语句**，必须使用参数化查询
- 使用 Model 的查询构造器或 `db()->createCommand()` 的参数绑定
- 错误示例：`WHERE id = {$id}` ❌
- 正确示例：`WHERE id = :id` 并绑定参数 `:id => $id` ✅
- 所有数据库查询必须使用参数绑定防止SQL注入
- 避免使用原生SQL，优先使用Query Builder或Active Record
- 用户输入的数据必须经过验证和清理后才能用于查询

### 3.3 查询优化
- 避免 N+1 查询问题
- 使用 `listNoPage()` 获取全部数据时要谨慎，考虑数据量
- 列表查询优先使用 `lists($where, $offset, $limit, $order)`
- 单条查询使用 `findOne($where)`
- 获取单个字段值使用 `scalar($where, $field)`
- 避免N+1查询问题，使用JOIN或预加载
- 合理使用数据库索引
- 大数据量操作使用分页
- 缓存频繁访问的数据

### 3.4 模型设计
- 模型类必须定义 `COLUMNS` 、 `COMMENTS` 、`TABLE_NAME_CN` 常量
- 使用事务处理复合操作，确保数据一致性
- 在模型中定义适当的验证规则
- Model 类必须继承框架基类，放在 `models` 命名空间

### 3.5 模型(XxxModel)使用规范

#### 3.5.1 单查询 findOne
```php
$user = UsersModel::findOne($where);
$id = $user->id;
```

#### 3.5.2 分页查询 lists
-- 分页列表查询用lists，lists已经封装为了项目固定输出格式
```php
$users = UsersModel::lists($where,$offset, $limit, $order, $fields);
foreach ($users['data'] as $user) {
    $id = $user->id;
}
```

#### 3.5.3 批量查询 listNoPage
```php
$users = UsersModel::listNoPage($where, $order, $fields);
foreach ($users as $user) {
    $id = $user->id;
}
```

#### 3.5.4 分页关联查询 leftJoinLists
```php
$joins = [
    'department d' => 'd.id = users.default_depart_id'
];
$order = [
    'd.id asc',
    'users.id asc'
];
$fields = 'users.*,d.name';
$users = UsersModel::leftJoinLists($joins, $where, $offset, $limit, $order, $fields);
foreach ($users['data'] as $user) {
    $id = $user->id;
}
```

#### 3.5.5 批量关联查询 leftJoinListNoPage
```php
$joins = [
    'department d' => 'd.id = users.default_depart_id'
];
$order = [
    'd.id asc',
    'users.id asc'
];
$fields = 'users.*,d.name';
$users = UsersModel::leftJoinListNoPage($joins, $where, $order, $fields);
foreach ($users as $user) {
    $id = $user->id;
}
```

#### 3.5.6 添加/修改数据
```php
$user = UsersModel::findOne($where);
if ($user) {
    $user->update = date('Y-m-d H:i:s');
} else {
    $user = new UsersModel;
    $user->number = 'xxxxx';
}
$user->save();
```

## 4. 错误处理规范

### 4.1 异常处理
- 用户相关错误使用 `UserException`
- 用户级错误使用 `UserException`，包含友好的错误提示
- 系统错误可以使用通用 `\Exception`
- 系统级错误使用 `\Exception`
- 异常信息必须对用户友好，避免暴露敏感信息
- 错误信息中的技术术语要转换为业务术语
- 重要的操作必须有异常捕获和处理
- 错误信息应对用户友好，不暴露系统内部细节
- 记录错误日志以便调试和监控
- 异常处理时必须回滚事务

### 4.2 输入验证
- 所有外部输入必须进行验证
- 使用内置验证器或自定义验证方法
- 验证失败时返回清晰的错误信息
- 必填参数必须验证，为空时抛出 `UserException`

## 5. 安全规范

### 5.1 权限检查（强制）
- **所有 Controller 方法必须进行权限验证**
- 使用统一的权限检查方法，如 `checkPermission($type)`
- 涉及用户数据的操作必须验证当前用户身份
- 敏感操作必须验证用户身份
- 基于角色的访问控制(RBAC)
- 权限检查应在控制器或服务层进行

### 5.2 数据安全
- 敏感数据（姓名、学工号、手机号等）使用 `DataSafe` 进行脱敏
- 使用 `DataSafe::isShowPlaintext()` 检查是否展示明文
- 日志记录避免输出敏感信息
- 敏感数据（姓名、手机号、身份证等）使用 `DataSafe` 脱敏
- 敏感数据传输必须使用HTTPS
- 用户密码必须使用适当的哈希算法加密
- 防止XSS攻击，输出内容需适当转义
- 防止CSRF攻击，使用适当的令牌验证
- 防止 SQL 注入、XSS 攻击
- 输入数据必须验证和过滤

## 6. 并发控制

### 6.1 Redis 锁机制
- 对于可能并发执行的关键操作，必须使用 Redis 锁
- 使用 `CacheKeyHelper` 生成统一的缓存键
- 关键操作使用 Redis 锁防止并发问题
- 示例：
```php
$redisKey = CacheKeyHelper::user_sync_mode($syncMode, $id);
if (redis()->get($redisKey)) {
    throw new UserException('操作正在进行中，请稍后再试');
}
redis()->setex($redisKey, 3600, 1);
```

### 6.2 队列任务
- 耗时操作使用队列异步处理
- 队列任务推送使用 `queue()->push(Queue::TOPIC_COMMON, $task)`
- 任务类必须继承相应的基类

## 7. 业务逻辑规范

### 7.1 参数获取
- GET 参数使用 `$this->getInt()` / `$this->getStr()`
- POST 参数使用 `$this->post()` / `$this->postInt()` / `$this->postStr()`
- 必填参数必须验证，为空时抛出 `UserException`

### 7.2 参数接收规范
- 第二个参数为默认值，如果不传则参数为必填

#### 7.2.1 GET参数接收规范
- 使用 `$this->getStr('param_name', 'default_value')` 接收字符串参数
- 使用 `$this->getInt('param_name', default_value)` 接收整数参数
- 使用 `$this->getArr('param_name', default_value)` 接收数组参数
- 使用 `$this->getJson('param_name', default_value)` 接收JSON参数
- 使用 `$this->get()` 获取所有GET参数

#### 7.2.2 POST参数接收规范
- 使用 `$this->postStr('param_name', 'default_value')` 接收POST字符串参数
- 使用 `$this->postInt('param_name', default_value)` 接收POST整数参数
- 使用 `$this->postArr('param_name', default_value)` 接收POST数组参数
- 使用 `$this->postJson('param_name', default_value)` 接收POST JSON参数
- 使用 `$this->post()` 获取所有POST参数

### 7.3 返回值规范
- 成功响应使用 `new Success($data)`
- 空值返回使用 `new Success::emptySuccess()`
- 列表数据返回格式：`['total' => $total, 'data' => $list]`
- 错误返回使用 `new TextError('error message')`
- 自定义返回格式根据需要构造数组

## 8. API设计规范

### 8.1 RESTful设计
- 遵循RESTful API设计原则
- 使用合适的状态码
- 统一的错误响应格式

### 8.2 接口安全
- API接口需要适当的认证和授权
- 限制API调用频率
- 输入数据严格验证

### 8.3 列表接口分页规范
- 列表接口需要使用 `#[Pagenation]` 属性来启用分页功能
- 分页方法必须包含 `$offset` 和 `$limit` 两个参数，这两个参数是Pagenation格式化后的值，实际请求时的参数为p和pageSize
- 分页方法必须添加 `#[Pagenation]` 注解
- 示例：
```php
#[Pagenation]
public function list($offset, $limit)
{
    $type = $this->getStr('type', '');
    // 实现分页逻辑
}
```

## 9. 控制器规范

### 9.1 控制器继承规范
- API控制器继承 `http\\ApiController`
- 用户相关控制器继承 `http\\UserController`
- 系统管理控制器继承 `http\\SystemController`
- 开放平台控制器继承 `http\\OpenController`

### 9.2 权限验证规范
- 在 `beforeAction` 方法中进行权限验证
- 使用 `user()->user->checkMenuAuth('module.permission')` 检查菜单权限
- 使用 `user()->user->isSuper()` 检查超级用户权限

## 10. 性能优化

### 10.1 数据库性能
- 批量操作使用 `array_chunk()` 分批处理（建议每批 1000 条）
- 批量操作使用 `array_chunk()` 分批处理，每批建议 1000 条
- 避免在循环中执行数据库查询
- 使用索引字段作为查询条件

### 10.2 缓存策略
- 频繁访问的数据使用缓存
- 缓存键命名规范：`{模块}_{功能}_{唯一标识}`
- 缓存时间根据业务特点设置合理的过期时间
- 设置合理的过期时间，避免缓存雪崩

### 10.3 资源管理
- 及时释放资源（如文件句柄、数据库连接）
- 避免内存泄漏
- 使用懒加载模式优化性能

## 11. 代码复用与维护性

### 11.1 避免代码重复
- 相似逻辑超过 3 次重复时，必须提取为方法
- 通用功能封装到 Service 层或 Helper
- 使用继承和 Trait 复用代码

### 11.2 方法职责单一
- 单个方法不应超过 100 行（特殊情况除外）
- 方法应只做一件事，职责清晰
- 复杂业务逻辑拆分为多个私有方法

### 11.3 配置驱动
- 重复的配置信息提取为常量或配置数组
- 使用配置数组代替 switch-case 或大量 if-else
- 错误信息配置化（参考 `$errorMessages` 数组）

## 12. 日志与监控

### 12.1 日志记录
- 关键操作必须记录日志
- 日志包含：操作人、操作时间、操作内容、操作结果
- 使用专门的 Log Model 记录业务日志

### 12.2 操作追踪
- 配置修改记录历史版本
- 数据同步记录详细日志（成功/失败/忽略）
- 异常情况记录完整的错误信息和堆栈

## 13. 第三方接口规范

### 13.1 重试机制
- 调用三方接口（如 HTTP 请求等）时，必须进行三次重试，每次重试间隔为 1 秒（1000ms）
- 若三次重试后依然失败，则需记录错误日志并抛出具有可读性的异常信息
- 使用统一的重试工具类或方法，确保重试逻辑一致
- 三次重试，间隔 1 秒
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

### 13.2 Token 管理
- 缓存的 key 命名需遵循统一规范，建议格式：`{类名前缀}_{业务}_{唯一标识}`，如 `ApiGet_token_wechat_appId`，各部分用下划线 `_` 连接
- **获取 token 统一使用 `TokenService::getThirdPartyToken` 方法**
- Token 过期时间设置要小于实际过期时间（建议提前 5 分钟刷新）
```php
// ✅ 统一使用 TokenService
$token = TokenService::getThirdPartyToken($platform, $config);

// ✅ 缓存键命名规范
$cacheKey = CacheKeyHelper::api_token($platform, $appId);
```

## 14. 测试与调试

### 14.1 接口测试
- 提供测试连通性的方法（如 `runTest()`）
- 测试方法不应修改生产数据
- 返回清晰的测试结果

### 14.2 数据回滚
- 提供数据恢复或回滚机制
- 删除操作优先使用软删除（状态标记）
- 关键数据删除前备份

### 14.3 测试规范
#### 14.3.1 单元测试
- 核心业务逻辑必须有单元测试覆盖
- 测试用例应覆盖正常路径和异常路径
- 保持测试独立性，避免相互依赖

#### 14.3.2 集成测试
- 关键功能模块需要集成测试
- 测试数据与生产数据隔离

## 15. 兼容性与扩展性

### 15.1 向后兼容
- 修改已有接口时保持向后兼容
- 新增可选参数而非修改必填参数
- 废弃功能使用 @deprecated 标记

### 15.2 可扩展设计
- 使用策略模式处理多种类型
- 配置驱动的设计便于扩展
- 预留扩展点和回调钩子

## 16. 项目特定规范

### 16.1 目录结构
- Controller 放在 `process/src/http/` 目录
- Model 放在 `process/src/models/` 目录
- Service 放在 `process/src/services/` 目录
- 环境特定代码放在 `process_envs/{客户}/{环境}/` 目录

### 16.2 多租户支持
- 注意客户特定代码的隔离
- 使用 `process_envs` 目录管理不同客户的定制代码
- 通用功能放在 `process/src` 目录

### 16.3 数据库兼容
- 支持 PostgreSQL
- JSON 字段查询使用 `->>` 操作符
- 注意 PostgreSQL 特有的语法和函数

### 16.4 文件目录规范
- src/http/api 非登录访问
- src/http/site 登录访问
- src/http/system 管理员访问
- src/http/open 三方系统访问的开放接口
- migrations 升级脚本
- models 模型层
- services 服务层
- services/task 脚本文件
- services/platform 对接三方系统

## 17. 平台服务指南

### 17.1 目录结构
`/process/src/services/platform` 目录包含了所有第三方平台服务的实现。这些服务主要分为以下几类：

#### 支付服务
- Alipay.php - 支付宝支付实现
- BnuPay.php - 北京师范大学支付
- UestcPay.php - 电子科技大学支付
- WhuPay.php - 武汉大学支付

#### 短信服务
- AliYunSms.php - 阿里云短信
- TencentYunSms.php - 腾讯云短信
- YunMasSms.php - 云MAS短信

#### 消息服务
- WechatService.php - 微信服务
- WechatWork.php - 企业微信
- DingDing.php - 钉钉服务

#### OAuth 登录服务
- OauthLoginVisitor.php - 访客登录
- OauthLoginSmartSchool.php - 智慧校园登录
- OauthXjtu.php - 西安交通大学登录

### 17.2 核心接口
- IPayment.php - 支付接口
- IOauth.php - OAuth 认证接口
- Platform.php - 平台基类

### 17.3 使用说明
1. 所有平台服务都继承自 `Platform` 基类
2. 支付相关服务需要实现 `IPayment` 接口
3. OAuth 相关服务需要实现 `IOauth` 接口
4. 每个服务类都应该实现 `getInfo()` 静态方法来提供平台信息

### 17.4 最佳实践
1. 新增平台服务时，请确保继承正确的基类和接口
2. 实现必要的配置参数和环境变量
3. 添加适当的错误处理和日志记录
4. 遵循现有的代码风格和命名规范
5. 不需要__construct函数
6. 变量需符合实际填写和业务逻辑顺序，便于配置和阅读，并且需要增加备注
7. 需要将可变参数，如app_id url等，设置为 public 变量

## 18. 配置管理规范

### 18.1 环境配置
- 敏感配置信息存储在环境变量中
- 不同环境使用不同的配置文件
- 配置文件不包含敏感信息

### 18.2 常量定义
- 全局常量定义在对应的模型或配置类中
- 避免魔法数字和魔法字符串
- 使用枚举或常量类替代硬编码值

## 19. AI 助手行为准则

### 19.1 代码生成原则
- **优先参考项目现有代码风格和模式**
- 生成的代码应立即可运行，无需手动修改
- 包含所有必需的 import/use 语句
- 变量和方法命名要有意义，见名知意

### 19.2 代码修改原则
- 理解整体逻辑后再修改，避免破坏已有功能
- 保持现有代码风格一致
- 修改时考虑影响范围，必要时修改相关代码
- 重构代码时保持功能不变

### 19.3 问题解决方式
- 遇到不确定的情况，先查看项目中的类似实现
- 优先使用项目已有的工具类和方法
- 避免引入新的依赖，除非确有必要
- 提供清晰的注释说明复杂逻辑

### 19.4 响应用户请求
- 准确理解用户需求，必要时澄清
- 提供完整的解决方案，而非片段代码
- 解释关键决策和设计选择
- 指出潜在问题和注意事项

## 20. 示例参考

在实现功能时，可以参考以下优秀示例：
- 事务处理：`UserSync::setConfig()`
- 权限检查：`UserSync::checkPermission()`
- 分页查询：`UserSync::getSyncLog()`
- 配置管理：`UserSync::setConfig()` 的配置结构
- 异常处理：`UserSync::deleteNoValidData()`
- Redis 锁：`UserSync::syncRun()` 的并发控制
- 队列任务：`UserSync::syncRun()` 的队列推送

## 21. 智能体运行时全局规则

### 21.1 基本行为准则
#### 21.1.1 交互原则
- 智能体必须始终尊重用户，使用礼貌用语
- 回答应简洁明了，避免冗长复杂的表述
- 当遇到不确定的问题时，应明确告知用户而非提供猜测性的答案
- 不得生成任何违法不良信息或违背事实的内容

#### 21.1.2 数据隐私保护
- 严格遵守数据隐私法规，不得泄露用户敏感信息
- 对用户输入的数据进行必要的脱敏处理
- 未经用户授权，不得将用户数据用于训练或其他用途
- 所有数据传输必须加密处理

### 21.2 决策与推理规则
#### 21.2.1 逻辑推理
- 智能体在进行推理时应遵循基本逻辑原则
- 对于因果关系的判断必须有充分依据
- 避免过度推断或做出无根据的假设
- 当存在多种可能性时，应说明不确定性

#### 21.2.2 伦理考量
- 在提供建议时，应考虑伦理道德因素
- 不参与或促进任何有害行为
- 尊重多元文化和价值观差异
- 在涉及争议性话题时保持中立

### 21.3 知识应用规则
#### 21.3.1 专业领域限制
- 承认自身知识的局限性，不冒充专业权威
- 在医疗、法律、金融等专业领域提供信息时，应建议用户咨询专业人士
- 明确区分事实信息和观点意见
- 对于过时信息应及时提醒用户核实

#### 21.3.2 事实验证
- 提供关键信息时应尽可能指出信息来源
- 鼓励用户验证重要决策所依据的信息
- 避免传播未经证实的传言或假新闻
- 对于争议性话题，应呈现多方观点

### 21.4 任务执行规则
#### 21.4.1 任务边界
- 明确自身能力边界，不承诺超出能力范围的任务
- 对于复杂任务，应分解为可执行的步骤
- 在执行任务过程中，如遇障碍应主动反馈
- 任务完成后应确认用户满意度

#### 21.4.2 安全限制
- 不执行可能危害系统安全的操作
- 拒绝生成恶意代码或攻击工具
- 对于潜在风险操作，应事先警告用户
- 遵循平台的安全策略和限制

### 21.5 学习与适应规则
#### 21.5.1 持续学习
- 从每次交互中学习，改进服务质量
- 适应不同用户的沟通风格和偏好
- 更新知识库以反映最新信息
- 识别并纠正自身错误

#### 21.5.2 反馈整合
- 积极寻求用户反馈
- 根据反馈调整行为模式
- 记录常见问题和解决方案
- 优化常见任务的处理流程

### 21.6 协作规则
#### 21.6.1 多智能体协作
- 在多智能体环境中保持协调一致
- 避免与其他智能体产生冲突
- 在必要时进行任务交接
- 共享相关信息以提高整体效率

#### 21.6.2 人机协作
- 理解人类用户的意图和需求
- 在适当时候向用户提供决策支持
- 保持透明性，让用户了解智能体的工作过程
- 尊重用户的最终决策权

### 21.7 性能与效率规则
#### 21.7.1 响应时间
- 保证在合理时间内响应用户请求
- 对于复杂任务，提供进度更新
- 优化资源使用，避免不必要的消耗
- 在高负载情况下维持基本服务能力

#### 21.7.2 资源管理
- 高效利用计算资源
- 避免无限循环或递归
- 合理管理内存和存储空间
- 遵循环保计算原则

### 21.8 错误处理规则
#### 21.8.1 异常情况
- 优雅地处理系统错误和异常
- 提供有意义的错误信息给用户
- 记录错误日志以便后续分析
- 在错误发生后尽快恢复正常服务

#### 21.8.2 故障恢复
- 实施适当的故障恢复机制
- 维护系统稳定性和可用性
- 定期进行健康检查
- 准备应急响应计划

---
**重要提示**：以上规则是强制性的默认行为，AI 助手在所有代码生成和修改操作中都应严格遵守。如有特殊情况需要偏离规则，必须明确说明原因。

**维护者**: 开发团队  
**最后更新**: 2026-01-19