# **代码review的原则**
1. 接口：面向领域、可读、版本、get/post、分页参数、token、req/res范围
2. 编码
    > 代码基础
        代码层次别太深
        vo(包括req/res)，DTO/DO，序列化标识
        sonar
        注释、日志（关键位置、关键信息、级别、动态级别）、错误码
        异常处理（是否处理，不要吃掉异常栈、处理范围、for循环中处理）
        事务、长事务、一致性、分布式锁、锁范围
    业务场景
        安全登录、登出、改密 
        上传下载（osskey规范、大文件、oss传输和存储中的域名规范）
3. 数据
    redis key,timeout
    mysql
4. 安全
    幂等、重放、蒙层
    参数校验（不信任调用方）、操控校验、trim
    id遍历、参数遍历（如登录场景码）
    id越权、参数越权：只认证未鉴权
    密码明文、密码复杂度、token未随机、验证码防刷、验证码失效策略、同一手机号验证码限制策略
    接口保护
    xss,csrf,springboot actuator
    日志审计
5. 可用
    异常处理
    容错处理（fallback）
    动态开关
    单点依赖的容错
6. 性能
    提前终止
    接口合并
    batch
    链路缩减
7. 灵活可配
    参数配置化
    配置动态化
	
	
Spring的缓存


#fallback

	/coop/queryCoopAuthecnticationByCoopeId 重试2次，失败直接抛出异常  超时时间3s 
	
	/coop/bankCard/submit  重试2次，失败直接抛出异常  超时时间6s 	
	
	/isCoopUserAvailbale 重试2次 ，失败直接抛出异常  超时时间3s 
	
	/isNickNameExit  商户内出单员名称不能重复（接口不合理）
	
	/antifraud/login 接受服务失败，suceess 
	
	/message/send/sms 发送短信失败 规则码怎么通知 ，能否临时生成验证码，当做短信校验
	
	/promotionActivity/apply  默认返回成功，走异步消息

	/queryDomainBySaasCodeAndSoure 重试2次  走缓存  超时时间3s 