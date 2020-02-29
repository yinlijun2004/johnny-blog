---
title: Spring Shiro多Realm登录实现
date: 2020-03-01 00:13:26
tags: [spring boot, shiro]
---

Spring Shiro是一款非常优秀的权限管理框架，之前用它来做用户的权限管理。最近碰到一个需求，就是需要兼容微信用户登录，之前已经介绍过如何在用[公众号登录](https://www.yinlijun.com/2020/02/29/微信第三方登录实现/)。那么，在公众号里登录的，肯定跟普通账户登录的权限一样（包括已绑定微信的账户）。

研究完Shiro的流程之后，发现做法也挺简单的，

### 实现AuthenticationToken接口
用来保存微信传过来的token
```
public class OAuth2Token implements AuthenticationToken {
    private String token;
    public OAuth2Token(String token){
        this.token = token;
    }

    @Override
    public Object getPrincipal() {
        return token;
    }

    @Override
    public Object getCredentials() {
        return token;
    }
}
```
### 获取token
实现一个AuthenticatingFilter接口的类，从ServletRequest中获取登录信息。
```
@Slf4j
public class OAuth2Filter extends AuthenticatingFilter {
    @Override
    protected AuthenticationToken createToken(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        String token = getToken((HttpServletRequest) servletRequest);
        if (StringUtils.isAllBlank(token)) {
            return null;
        }
        //返回一个token实例
        return new OAuth2Token(token);
    }

    @Override
    protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        String token = getToken((HttpServletRequest) servletRequest);
        if (StringUtils.isBlank(token)) {
            //无效token处理
        }
        return executeLogin(servletRequest, servletResponse);
    }

    private String getToken(HttpServletRequest request){
        return request.getHeader("token");

    }
}

```

### 将OAuth2Filter配置到ShiroFilterFactoryBean
```java
@Bean
public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
    ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
    shiroFilterFactoryBean.setSecurityManager(securityManager);

    //oauth过滤
    Map<String, Filter> filters = new HashMap<>();
    filters.put("oauth2", new OAuth2Filter());
    shiroFilterFactoryBean.setFilters(filters);
    ...
}
```
### 继承AuthorizingRealm类，处理token，并授权
```java
@Slf4j
@Component("weixinRealm")
public class WeixinRealm extends AuthorizingRealm {
    @Override
    public boolean supports(AuthenticationToken token) {
        //判断token是否我们要处理的token类型
        return token instanceof OAuth2Token;
    }

    /**
     * 授权模块，获取用户角色和权限
     *
     * @param principal principal
     * @return AuthorizationInfo 权限信息
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principal) {

        Object obj = principal.getPrimaryPrincipal();
        //这个obj就是主体
        //判断主体是否是当前需要的用户类型。如果不是这个Realm能处理的主体，直接返回null就好
        //构造权限信息。
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();

        // 根据主体获取角色集合和权限集合
        Set<String> roleSet = null；
        simpleAuthorizationInfo.setRoles(roleSet);
        Set<String> permissionSet = null
        simpleAuthorizationInfo.setStringPermissions(permissionSet);

        return simpleAuthorizationInfo;
    }

    /**
     * 用户认证
     *
     * @param auth AuthenticationToken 身份认证 token
     * @return AuthenticationInfo 身份认证信息
     * @throws AuthenticationException 认证相关异常
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken auth) throws AuthenticationException {
        //这个token就是刚刚保存的token
        String token = (String) auth.getCredentials();
        //根据token解出userId.
        //通过userId找出user
        //将user对象传给SimpleAuthenticationInfo作为主体(principal)
        return new SimpleAuthenticationInfo(user, token, getName());
    }
}
```
### 多realm处理
生成SecurityManager的bean，将我们的realm列表注入进去。
```java
 @Bean
public SecurityManager securityManager() {
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    // 配置 SecurityManager，并注入 shiroRealm
    Collection<Realm> realms = Arrays.asList(realm1, realm2, realm3);
    securityManager.setRealms(realms);
    // 配置 rememberMeCookie
    securityManager.setRememberMeManager(rememberMeManager());
    // 配置 缓存管理类 cacheManager
    securityManager.setCacheManager(cacheManager());
    securityManager.setSessionManager(sessionManager());
    return securityManager;
}
```