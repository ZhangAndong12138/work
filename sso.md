# labor-screen、BiReport与bsp平台集成步骤

~~注意！：由于目前BiReport系统存在bug，在进行集成开始前请添加QQ `4000011866`，咨询编号为`ESENBI-13417`的bug是否已解决。如未解决，请停止集成。~~ 此问题已解决。

## 一、labor-screen项目相关配置

1. 更新项目代码，确保labor-screen项目代码为最新并没有编译错误、`web.xml`中单点登录相关的filter配置没有被注释
2. 修改项目配置文件

>/src/main/resources/security.properties

把`sso.saml.idp.sso`、`bsp.restful.server` 修改为bsp平台正式环境的ip和端口

把`sso.saml.sp.assertionconsumer` 改为labor-screen项目正式环境的ip和端口

>/src/main/resources/url.properties

把`labor-screen`、`labor-bireport`、`labor-cloudso` 改为各自对应项目正式环境的ip和端口

~~或直接注释掉[开发环境]/[测试环境]部分，并解除[正式环境]部分的注释~~

3. 修改`sso_es-1.2.0.jar`配置文件

>/src/lib/sso_es-1.2.0.jar/META-INF/esensso/ssoconfig-master.xml

需要使用WinRAR或其它解压工具打开`sso_es-1.2.0.jar`，然后按路径找到文件 ~~直接在IDEA中打开文件并不能修改~~，把`login-pageurl`、`logout-pageurl`改为BiReport系统正式环境的ip和端口

4. 打包labor-screen项目

打包完成后请使用WinRAR或其他解压工具确认`WEB-INF/lib/` 路径下有`sso_es-1.2.0.jar`，并确认`sso_es-1.2.0.jar`中的`ssoconfig-master.xml`已经按照*一.3*中的步骤修改完成

5. 部署

## 二、BiReport相关配置
1.  创建用户和机构视图

请求浪潮DBA协助在正式环境创建bi_user和bi_organ视图，需要执行以下SQL：

```sql
-- bi_user 用户视图
CREATE OR REPLACE VIEW bi_user AS 
SELECT DISTINCT
	u.user_id AS user_id,
	u.user_name AS user_name,
	u.password AS password,
	o.organ_code AS organ_code 
FROM
	( ( pub_users u JOIN pub_user_employee e ) JOIN pub_organ o ) 
WHERE
	(( o.stru_level < 4 ) 
	AND ( u.user_id = e.USER_ID ) 
	AND ( e.DEPARTMENT_ID = o.organ_id ) );
```
```sql
-- bi_organ 机构视图
CREATE OR REPLACE VIEW bi_organ AS
SELECT
	a.organ_code AS organ_code,
	a.organ_name AS organ_name,
	(CASE WHEN isnull( ( SELECT b.organ_code FROM pub_organ b WHERE ( a.parent_id = b.organ_id ) ) ) THEN
	'rootid' ELSE ( SELECT b.organ_code FROM pub_organ b WHERE ( a.parent_id = b.organ_id ) ) END 
	) AS parent_code 
FROM
	pub_organ a 
WHERE
( a.stru_level < 4 );
```

2. 在BiReport系统中配置用户机构数据源

使用**管理员账户**登录BiReport系统，进入`系统管理->数据库连接池`，新建一个连接池或克隆现有连接池，
- `名称`：自定 ~~比如labor_user~~
- `描述`：如实填写
- `数据库类型`、`链接地址`、`用户名`、`密码`：请向浪潮DBA咨询后如实填写。

**保存前请确保测试通过**

3. BiReport连接用户机构库

使用**管理员账户**登录BiReport系统，进入`用户权限->库表配置`，

- `机构模式`：第三方机构库表
- `数据库连接池`：选择*二.2*中配置的数据库连接池名称 ~~比如labor_user~~
- `用户表名`：bi_user
- `机构表名`：bi_organ
- `根机构代码`：rootId

点击*下一步*

用户信息对应字段
- `用户代码`：user_id
- `用户名称`：user_name
- `密码`：password
- `所保存密码的加密形式`：没有加密
- `机构代码`：organ_code

 机构信息对应字段
- `机构代码`：organ_code
- `机构名称`：organ_name
- `上级机构代码`：parent_code

点击*测试*，并确保通过**全部六项测试**后，点击*完成*，根据提示重启服务器

4. 修改BiReport单点登录配置文件

从BiReport正式环境服务器上获取BiReport的war包，并修改其中的
>WEB-INF/classes/META-INF/esensso/ssoconfig-slave.xml

把`master-action`、`login-error-page`改为labor-screen项目正式环境的ip和端口

把`ssoconfig`中的`sysid`设置为`sysid="esenface"`

5. 修改BiReport的war包中WEB-INF/web.xml文件，在适当位置添加如下内容：
```xml
<filter>
    <filter-name>checkLoginFilter</filter-name>
    <filter-class>com.esen.util.sso.slave.CheckLoginFilter</filter-class>
    <init-param>
        <param-name>notCheckURLList</param-name>
        <param-value>
            /esmain/login.do;/esmain/portal/loginportal.do;/esmain/portal/portalmgr.do;/esmain/setup.do;/esmain/verifycode.do;/esmain/register.do;/resource/ES$8$regserver;/esmain/setupserver.do;/esmain/error.do;/esmain/locale.do;/test.do;/i18n.do
        </param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>checkLoginFilter</filter-name>
    <url-pattern>*.do</url-pattern>
</filter-mapping>
```
6. 确认*二.4*和*二.5*修改的文件都正确保存后，进入`系统管理->性能与维护->系统维护->重启服务器`

## 三、cloud_so项目相关配置

1. 更新项目代码，确保cloud_so项目代码为最新并没有编译错误、`web.xml`中单点登录相关的filter配置没有被注释
2. 修改项目配置文件

>/src/main/resources/security.properties

把`sso.saml.idp.sso`、`bsp.restful.server` 修改为bsp平台正式环境的ip和端口

把`sso.saml.sp.assertionconsumer` 改为cloud_so项目正式环境的ip和端口

3. 打包
4. 部署




