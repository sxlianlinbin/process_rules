---
description: 
globs: 
alwaysApply: false
---
# 平台服务目录指南

## 目录结构

`/process/src/services/platform` 目录包含了所有第三方平台服务的实现。这些服务主要分为以下几类：

### 支付服务
- [Alipay.php](mdc:process/src/services/platform/Alipay.php) - 支付宝支付实现
- [BnuPay.php](mdc:process/src/services/platform/BnuPay.php) - 北京师范大学支付
- [UestcPay.php](mdc:process/src/services/platform/UestcPay.php) - 电子科技大学支付
- [WhuPay.php](mdc:process/src/services/platform/WhuPay.php) - 武汉大学支付

### 短信服务
- [AliYunSms.php](mdc:process/src/services/platform/AliYunSms.php) - 阿里云短信
- [TencentYunSms.php](mdc:process/src/services/platform/TencentYunSms.php) - 腾讯云短信
- [YunMasSms.php](mdc:process/src/services/platform/YunMasSms.php) - 云MAS短信

### 消息服务
- [WechatService.php](mdc:process/src/services/platform/WechatService.php) - 微信服务
- [WechatWork.php](mdc:process/src/services/platform/WechatWork.php) - 企业微信
- [DingDing.php](mdc:process/src/services/platform/DingDing.php) - 钉钉服务

### OAuth 登录服务
- [OauthLoginVisitor.php](mdc:process/src/services/platform/OauthLoginVisitor.php) - 访客登录
- [OauthLoginSmartSchool.php](mdc:process/src/services/platform/OauthLoginSmartSchool.php) - 智慧校园登录
- [OauthXjtu.php](mdc:process/src/services/platform/OauthXjtu.php) - 西安交通大学登录

## 核心接口

- [IPayment.php](mdc:process/src/services/platform/IPayment.php) - 支付接口
- [IOauth.php](mdc:process/src/services/platform/IOauth.php) - OAuth 认证接口
- [Platform.php](mdc:process/src/services/platform/Platform.php) - 平台基类

## 使用说明

1. 所有平台服务都继承自 `Platform` 基类
2. 支付相关服务需要实现 `IPayment` 接口
3. OAuth 相关服务需要实现 `IOauth` 接口
4. 每个服务类都应该实现 `getInfo()` 静态方法来提供平台信息

## 最佳实践

1. 新增平台服务时，请确保继承正确的基类和接口
2. 实现必要的配置参数和环境变量
3. 添加适当的错误处理和日志记录
4. 遵循现有的代码风格和命名规范
5. 不需要__construct函数
6. 变量需符合实际填写和业务逻辑顺序，便于配置和阅读，并且需要增加备注
7. 需要将可变参数，如app_id url等，设置为 public 变量

