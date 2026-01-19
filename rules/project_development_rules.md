---
trigger: always_on
alwaysApply: true
---
# 项目开发全局规则

## 1. 代码规范

### 1.1 命名规范
- 类名使用帕斯卡命名法（PascalCase），如 `UserController`, `MapConfigModel`
- 方法名使用驼峰命名法（camelCase），如 `getUserInfo`, `validateInput`
- 常量使用全大写下划线分隔，如 `TYPE_LOGIN_CONFIG`, `STATUS_ACTIVE`
- 变量名使用驼峰命名法，具有描述性，避免单字母变量（循环变量除外）

### 1.2 代码格式
- 使用4个空格作为缩进，不允许使用Tab
- 每行代码不超过120个字符
- 类的左大括号 `{` 必须放在方法声明的下一行
- 控制结构关键字(if, for, while等)后必须有一个空格
- 控制结构的左大括号 `{` 必须跟关键字在同一行

### 1.3 注释规范
- 类必须包含PHPDoc注释，描述类的功能和作者信息
- 公共方法必须有PHPDoc注释，包含参数、返回值和异常说明
- 复杂逻辑块前添加行内注释解释其作用

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

### 3.1 查询安全
- 所有数据库查询必须使用参数绑定防止SQL注入
- 避免使用原生SQL，优先使用Query Builder或Active Record
- 用户输入的数据必须经过验证和清理后才能用于查询

### 3.2 模型设计
- 模型类必须定义 `COLUMNS` 、 `COMMENTS` 、`TABLE_NAME_CN` 常量
- 使用事务处理复合操作，确保数据一致性
- 在模型中定义适当的验证规则

## 4. 错误处理规范

### 4.1 异常处理
- 控制器方法异常响应使用UserException
- 服务层异常响应使用 Exception
- 重要的操作必须有异常捕获和处理
- 错误信息应对用户友好，不暴露系统内部细节
- 记录错误日志以便调试和监控

### 4.2 输入验证
- 所有外部输入必须进行验证
- 使用内置验证器或自定义验证方法
- 验证失败时返回清晰的错误信息

## 5. 安全规范

### 5.1 认证与授权
- 敏感操作必须验证用户身份
- 基于角色的访问控制(RBAC)
- 权限检查应在控制器或服务层进行

### 5.2 数据安全
- 敏感数据传输必须使用HTTPS
- 用户密码必须使用适当的哈希算法加密
- 防止XSS攻击，输出内容需适当转义
- 防止CSRF攻击，使用适当的令牌验证

## 6. 性能规范

### 6.1 查询优化
- 避免N+1查询问题，使用JOIN或预加载
- 合理使用数据库索引
- 大数据量操作使用分页
- 缓存频繁访问的数据

### 6.2 资源管理
- 及时释放资源（如文件句柄、数据库连接）
- 避免内存泄漏
- 使用懒加载模式优化性能

## 7. 测试规范

### 7.1 单元测试
- 核心业务逻辑必须有单元测试覆盖
- 测试用例应覆盖正常路径和异常路径
- 保持测试独立性，避免相互依赖

### 7.2 集成测试
- 关键功能模块需要集成测试
- 测试数据与生产数据隔离

## 9. 配置管理规范

### 9.1 环境配置
- 敏感配置信息存储在环境变量中
- 不同环境使用不同的配置文件
- 配置文件不包含敏感信息

### 9.2 常量定义
- 全局常量定义在对应的模型或配置类中
- 避免魔法数字和魔法字符串
- 使用枚举或常量类替代硬编码值

## 10. API设计规范

### 10.1 RESTful设计
- 遵循RESTful API设计原则
- 使用合适的状态码
- 统一的错误响应格式

### 10.2 接口安全
- API接口需要适当的认证和授权
- 限制API调用频率
- 输入数据严格验证

## 11. 控制器与参数规范

### 11.1 列表接口分页规范
- 列表接口需要使用 `#[Pagenation]` 属性来启用分页功能
- 分页方法必须包含 `$offset` 和 `$limit` 两个参数，这两个参数是Pagenation格式化后的值，实际请求时的参数为p和pageSize
- 示例：
```php
#[Pagenation]
public function list($offset, $limit)
{
    $type = $this->getStr('type', '');
    // 实现分页逻辑
}
```

## 12. 参数接收规范
- 第二个参数为默认值，如果不传则参数为必填

### 12.1 GET参数接收规范
- 使用 `$this->getStr('param_name', 'default_value')` 接收字符串参数
- 使用 `$this->getInt('param_name', default_value)` 接收整数参数
- 使用 `$this->getArr('param_name', default_value)` 接收数组参数
- 使用 `$this->getJson('param_name', default_value)` 接收JSON参数
- 使用 `$this->get()` 获取所有GET参数

### 12.3 POST参数接收规范
- 使用 `$this->postStr('param_name', 'default_value')` 接收POST字符串参数
- 使用 `$this->postInt('param_name', default_value)` 接收POST整数参数
- 使用 `$this->postArr('param_name', default_value)` 接收POST数组参数
- 使用 `$this->postJson('param_name', default_value)` 接收POST JSON参数
- 使用 `$this->post()` 获取所有POST参数


## 13 控制器继承规范
- API控制器继承 `http\\ApiController`
- 用户相关控制器继承 `http\\UserController`
- 系统管理控制器继承 `http\\SystemController`
- 开放平台控制器继承 `http\\OpenController`

## 14 权限验证规范
- 在 `beforeAction` 方法中进行权限验证
- 使用 `user()->user->checkMenuAuth('module.permission')` 检查菜单权限
- 使用 `user()->user->isSuper()` 检查超级用户权限

## 15 返回值规范
- 成功返回使用 `new Success($data)`
- 空值返回使用 `new Success::emptySuccess()`
- 错误返回使用 `new TextError('error message')`
- 自定义返回格式根据需要构造数组

## 16 模型(XxxModel)使用规范

### 16.1 单查询 findOne
```php
$user = UsersModel::findOne($where);
$id = $user->id;
```

### 16.2 分页查询 lists
-- 分页列表查询用lists，lists已经封装为了项目固定输出格式
```php
$users = UsersModel::lists($where,$offset, $limit, $order, $fields);
foreach ($users['data'] as $user) {
    $id = $user->id;
}
```

### 16.3 批量查询 listNoPage
```php
$users = UsersModel::listNoPage($where, $order, $fields);
foreach ($users as $user) {
    $id = $user->id;
}
```

### 16.4 分页关联查询 leftJoinLists
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

### 16.4 批量关联查询 leftJoinListNoPage
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

### 16.5 添加/修改数据
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

## 17 命名空间规范

- 核心代码 process/src/ 目录下的类文件命名空间从 src 后面的目录开始
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
- 定制化代码 process_envs/cust/env 目录下的类文件命名空间
    1、env 目录下的子目录固定为 product 和 test
    2、命名空间从 env 后面的目录开始
    3、命名空间前面固定增加一层 envs
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

## 18 文件目录规范
- src/http/api 非登录访问
- src/http/site 登录访问
- src/http/system 管理员访问
- src/http/open 三方系统访问的开放接口
- migrations 升级脚本
- models 模型层
- services 服务层
- services/task 脚本文件
- services/platform 对接三方系统

---

*本规则自发布之日起生效，可根据项目发展实际情况进行修订。*