## **账号手机号改造**

### 改造点
	1. 登录校验器开发 ok
	2. 注册信息流程改造 ok /verifyCode/register
	3. 创建账号分布式锁 
	4. 密码校验后台校验 ok
	5. password_salt的处理
	6. 海报拉新
	7. 手机号需对100的进行特殊处理
	8. 推广位号/手机号 展示调整

### 待解决

1. 出单员创建时，此时如果没有手机号 如何编辑 ？ 查询列表 怎么展示手机号 和 账户；创建是 默认只能有手机号；

2. 前端涉及的改动：

出单员信息查询： /subao-proxy-customer/coopUser/pageQuery acount 换为 phone
商户官网注册： /subao-proxy-customer/verifyCode/register 参数不变
删除： subao-customer/coop/info/isAldAccount

### 发布计划

#### 数据库

-- 设置密码不为空的passwordSalt为手机号
```
UPDATE subao_cooperator_user SET password_salt = phone WHERE PASSWORD IS NOT NULL AND PASSWORD != '';
```
-- 清理手机号为100的数据
```
UPDATE subao_cooperator_user SET phone = '' WHERE phone like '100%' and account like '100%';
```
-- 清空手机号存在的账号信息
```
UPDATE subao_cooperator_user SET account = '' WHERE phone not like '100%' and account = phone;
```

#### customer新增配置
```
subao.web.publicKey= MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCWwmOYs9ZbmnCXu/vj1jEUsMg0RQ5HMzgBCPXzd/hHk2nFI8J1/gcYykHVVltotKfDnxQb3XlUCQjYxkrpN5335ncF/SaaUKjGClnyrUaAZmaNd4nV+twnV8u3MUs4HUiC9XWmGjTpJKULEc5LGeNGo5weFQJbK8Hq94dWfbHncQIDAQAB
subao.web.privateKey=MIICdgIBADANBgkqhkiG9w0BAQEFAASCAmAwggJcAgEAAoGBAJbCY5iz1luacJe7++PWMRSwyDRFDkczOAEI9fN3+EeTacUjwnX+BxjKQdVWW2i0p8OfFBvdeVQJCNjGSuk3nffmdwX9JppQqMYKWfKtRoBmZo13idX63CdXy7cxSzgdSIL1daYaNOkkpQsRzksZ40ajnB4VAlsrwer3h1Z9sedxAgMBAAECgYAsnsollwyZviMW9b9+1pVeP1hyCAJK1oD75XAOKHLmlF3YyFW04IOvNaW4g7+4RMfmoT2tZaaUVbw1lLL1Rc3N7grD96+kjF6A3JK05GE0MVnLUnUDkt3EK5TSb0QSlaE93jPjEGUoiMGaRMyN2KHIyj2EnvOM+qiB2YXvpKWLNQJBAM6h+Kwga667d4k6Hm3BKfd7+88j3O5V80miU3JlMq0TQmV02oza9bnXZhq3sa4Zs2mY3b1ApHljklQ6Fk8O+XcCQQC6xxoIiHtq+F0MKCn+bW27xJb1xfX9FAO07j/iw4Yb1gXMrVq4LPkoz9keOPHpHqSv59dSHnFxzEPWa40KBeBXAkB+WMneLgDKGfUQosoCYG+R1Wz2jr5DuaiGaMxPkZN4AuPBd78/C4/fL9dOFO5/P1XWBtPnKcSoOPs7pz0G4GgnAkEAtZRUodEVswPraaMuWDRIRxAy9pLSt4waom4w26LcIPmrv7UUihLm059lY7VWsRziXETXFvWTsl8z4CPFBOZ7lwJAMCDkZd825ZgVMuNLZQfWOGwXCChBOrnw7m/tjU1Pwwr/vETRsBkrK584arF8jZv17IhkirOBXJnhEBAJReDs7w==
```