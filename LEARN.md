2020.03.06
简介：server_eggjs（商城、商店批发或零售，pc管理端 + 微信小程序 + 后端服务）
技术栈：egg2.26 + egg-passport + egg-jwt + passport-local + egg-redis + egg-mysql + egg-squelize + egg-session-redis
git地址：https://github.com/ruiyong-lee/weapp-vue-eggjs-shop-demo
使用的组件：无
优势：

eggjs执行过程：router ==> controller ===> service ===> model ===> sequelize

1、.autod.conf.js
autod ----Auto generate dependencies and devDependencies by parse the project file.
疑似egg-ci添加的。

2、app.js
app.js 和 agent.js 用于自定义启动时的初始化工作，可选
https://eggjs.org/zh-cn/basics/app-start.html

3、config
1）config.default.js
csrf：https://eggjs.org/zh-cn/core/security.html
keys
mysql
sequelize
io
redis
sessionRedis
jwt
middleware
2）plugin.js 开启各种插件
mysql
sequelize
redis
sessionRedis
validate
jwt
passport
io
那么，passport和jwt是什么关系呢？

4、app/router.js
微信小程序
管理端：
账户相关、
消息通知、
管理员
商家：
订货单、
商品、
商品类别、
运费方案、
送货时间、
统计、
utils、
soket.io
5、app/controller
1）BaseController
基础类实现了只读属性user、success、fail、noFound方法。（在新项目中，我把这些方法扩展到了app/extend/context.js上。因为RESTful风格api没办法用类式控制器）
2）controller/weapp.js
关于小程序的登录login方法，用了app._.isEmpty判断商家是否存在。lodash绑定在application上。
小程序商家登录的流程：用商家的appId，appSecret和code获取登录凭证校验。并将登录凭证保存在redis里。
// 登录凭证校验
const weappInfo = await ctx.curl(`https://api.weixin.qq.com/sns/jscode2session?appid=${merchant.appId}&secret=${merchant.appSecret}&js_code=${code}&grant_type=authorization_code`, {
  dataType: 'json',
}) || {};
const { openid: openId, session_key } = weappInfo.data || {};
if (openId) {
  const result = JSON.stringify({ openId, session_key });
  // 保存openId和session_key到redis
  await app.redis.get('default').setex(sessionid, 3600 * 24, result);
} else {
  return this.fail(ctx.ERROR_CODE, weappInfo.data.errmsg);
}

3）controller/user/
登录login，直接查用户的账号密码是否存在数据库里。若存在用户，设置token（扩展方法，种植cookie）（ctx.setToken）
注销logout，ctx.removeToken
修改密码接口、新增商家、修改商家、商家列表、商家详情
4）controller/utils/
upload.js（不好）
5）controller/notice.js 消息通知
全部标记成已读、获取消息概况、消息列表
6）controller/statistics.js 统计
获取今日订单数量统计、获取近7日订单统计
7）controller/freight_plan.js 运费方案
新增运费方案、修改运费方案、删除运费方案、运费方案列表、运费方案详情、设置默认运费方案
8）controller/delivery_time_type.js 送货时间
新增送货时间、修改送货时间、删除送货时间、送货时间列表、送货时间详情
9）controller/goods.js 商品
新增商品、修改商品、上架商品、下架商品、商品列表、商品详情、
10）controller/goods_category.js 商品列表
新增类别、修改类别、删除类别、类别列表、类别下拉列表、类别详情
11）controller/goods_order.js 订货单
订单列表、订单详情、配送订单、完成订单

6、app/service
1）service/user 
查找某个管理员。app.mysql.get（账号，密码用md5包加密）查找
修改管理员密码。savePasswordModify是在model上写的。
商家信息的增删改查，地址的增删改查，通过squelize在model上操作的。

7、app/model
1）model/user 
通过schema定义model，并在model上定义了一些方法。有2处连表查询
到底需不需要连表查询呢？

8、app/middleware
1）middleware/auth.js
微信小程序路由，/weapp/，通过sessionId查找redis库拿到openId。
管理后台路由，忽略login和logout。ctx的扩展方法verifyToken检查cookie合法性。流程是：
this.getAccessToken拿到token（扩展方法，通过cookie拿到token），app.jwt.verify校验token合理性。如果错误是TokenExpiredError，则重新setToken。并返回token校验结果为真。如果没有错误，返回token校验结果为真。其它情况返回token校验结果为假。在TokenExpiredError过期时能够从cookie里拿到user信息。这里就是jwt的理念：所有用户信息保存在客户端。
2）middleware/error_handler.js
在所有后续的next()上加一个try catch捕获错误。看起来不错！

9、app/extend
1）extend/application.js
lodash、日期格式化、基于symbol创建事务、删、单号生成、检查update、count是否成功、添加订阅延迟任务。
2）extend/context.js
状态码常量、获取token、设置token、移除token、校验token
3）extend/helper.js
字符串转JSON，JSON.parse。socket.io封装
10、app/io
用户消息通知

11、app/public
用户图片上传
12、app/schema
用于定义数据库字段
