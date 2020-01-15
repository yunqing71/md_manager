#一、前言
>本文根据我的项目进行Security密码认证的源码级别讲解，我们将通过localhost:9090访问来开始进行Debug说明，我已经在源码中打了很多个端点，基本能讲到Security用户名密码认证的全部流程，主要是给自己加深印象，其次分享给大家，如果讲解过程中有什么错误，也请大家不吝指正，谢谢！代码基于springboot2.2.1、security5、jdk8、mysql8.0、maven构建。
 
#二、debug启动springboot应用
##1、由于我们在MvcConfig配置文件中进行如下配置，所以访问localhost:9090会跳转home.html
```
/**
 * @author yunqing
 */
@Configuration
public class MvcConfig implements WebMvcConfigurer {

	@Override
	public void addViewControllers(ViewControllerRegistry registry) {
		registry.addViewController("/home").setViewName("home");
        //注意：这里配置了 / 跳转home.html页面
		registry.addViewController("/").setViewName("home");
		registry.addViewController("/hello").setViewName("hello");
		registry.addViewController("/login").setViewName("login");
	}

}
```
###1.1、application.yml中配置了端口为9090
```
server:
  port: 9090
```
###1.2、我在WebSecurityConfig中配置了不需要认证就可以访问的页面，其中包含 / 和/home
```
@Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                // 所有用户均可访问的资源
                .antMatchers("/css/**", "/js/**","/images/**", "/webjars/**", "**/favicon.ico", "/index").permitAll()
                .antMatchers(HttpMethod.POST, "/user/registration").permitAll()
                .antMatchers("/", "/home","/user/registration","/hello").permitAll()
                //剩下的任何请求都需要进行认证
                .anyRequest().authenticated()
                .and()
                //表单登录
                .formLogin()
                //登录请求页面
                .loginPage("/login")
                //自定义登录成功和失败处理器
                .successHandler(ajaxAuthSuccessHandler)
                .failureHandler(ajaxAuthFailHandler)
                .permitAll()
                .and()
                .logout()
                .permitAll();
    }
```
##2、看完上面的基本介绍，接下来我们进入了第一个断点
![image.png](https://upload-images.jianshu.io/upload_images/8020666-7b9dc56fc5cfc02b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>可以看到进了断点停在了抽象类AbstractAuthenticationToken,并且权限信息传入ROLE_ANONYMOUS参数，匿名角色，按下一步发现进入了如下AnonymousAuthenticationToken类，可以发现是anonymousUser匿名用户封装了一个匿名认证的Token，通过this.setAuthenticated(true0)设置认证通过。


![image.png](https://upload-images.jianshu.io/upload_images/8020666-ed92d1bc2e758e42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>所以得出结论：WebSecurityConfig中我们设置不需认证的资源或者路径，实际上Security使用匿名用户进行认证访问。

###2.1、这里扩展一下spring security过滤器链
![image.png](https://upload-images.jianshu.io/upload_images/8020666-c39a5f9e6b339e2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>如上图所示：SecurityContextPersistenceFilter过滤器位于过滤器链的最前端，请求先进这个过滤器，当请求进入，检查session中是否有securityContext,如果有则拿出来放到线程中，请求响应回来最后一个也经过此过滤器，检查线程中是否有SecurityContext，如果有则拿出来存到Session中。这样就可以完成认证结果在多个请求之间共享。

###2.2、通过获取共享在多个请求之间的用户信息
```
/**
 * @author yunqing
 * @Date 2019/12/15 16:44
 */
@Slf4j
@RestController
@RequestMapping("/api/account")
public class SecurityController {

    @GetMapping("/me")
    public Object getAuthentication() {
        return SecurityContextHolder.getContext().getAuthentication();
    }
}
```
>记得在SecurityConfig中设置/api/account/me不需要认证，即匿名登录。

![image.png](https://upload-images.jianshu.io/upload_images/8020666-b9ac610e0fc3bcdb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###2.3、获取当前认证用户信息
>直接跳过断点localhost:9090访问到 home.html页

![image.png](https://upload-images.jianshu.io/upload_images/8020666-f96b45a38555e581.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>访问localhost:9090/api/account/me获取当前认证用户信息，可以看到当前确实是匿名用户

```
#返回结果，证明当前认证成功的是匿名用户

{"authorities":[{"authority":"ROLE_ANONYMOUS"}],"details":
{"remoteAddress":"0:0:0:0:0:0:0:1","sessionId":"45C0AA46462B773A0606D24C91D70722"},
"authenticated":true,"principal":"anonymousUser","keyHash":431445726,"credentials":"",
"name":"anonymousUser"}
```
##3、正式开始讲解数据库中的用户认证
![image.png](https://upload-images.jianshu.io/upload_images/8020666-86584d0331ac995c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###3.1、点击Sign In登录跳转到第一个断点UsernamePasswordAuthenticationFilter
![image.png](https://upload-images.jianshu.io/upload_images/8020666-7d8621da47461980.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


>断点停到UsernamePasswordAuthenticationFilter，判断当前请求是否是POST请求→获取表单提交的用户密码→根据用户名密码构造一个UsernamePasswordAuthenticationToken→进入我们发现两个构造函数如下图

![image.png](https://upload-images.jianshu.io/upload_images/8020666-ef53fabf930fca20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>可以发现走的第一个两个参数的构造函数，因为并没有传入authorities角色信息，通过this.setAuthenticated(false)设置未经过认证→返回未认证的token到UsernamePasswordAuthenticationFilter

![image.png](https://upload-images.jianshu.io/upload_images/8020666-cace1b37a3e3de1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>通过this.setDetails(request,authRequest);设置ip、sessionId等信息到UsernamePasswordAuthenticationToken中，如下图所示：

![image.png](https://upload-images.jianshu.io/upload_images/8020666-b9f2c2c79779c454.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>返回UsernamePasswordAuthenticationToken到this.getAuthenticationManager().authenticate()进行处理,如下图所示：

![image.png](https://upload-images.jianshu.io/upload_images/8020666-50fcbbf2a7d3cd93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>之后进入ProviderManager中的断点，介绍一下，ProviderManager实现了AuthenticationManager接口，可以看到如下图，var8是一个Providers集合，循环遍历这个集合，找到适合处理UsernamePassword的Provider进行处理用户名密码认证，可以看到当前是处理匿名用户的AnonymousAuthenticationProvider，继续下一步

![image.png](https://upload-images.jianshu.io/upload_images/8020666-7450491cbe7aeab1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>循环直到当前Provider是DaoAuthenticationProvider这个是专门处理用户名密码认证的Provider,然后执行到result = provider.authenticate(authentication)；这个断点的时候，会进入到AbstractUserDetailsAuthenticationProvider 抽象类，它实现了 AuthenticationProvider接口，而DaoAuthenticationProvider又继承了这个抽象类。

![image.png](https://upload-images.jianshu.io/upload_images/8020666-bb8afa1ba4552490.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>接下来重点讲解DaoAuthenticationProvider和AbstractUserDetailsAuthenticationProvider 这两个类，主要的认证方法写在了这个抽象类中，如下图断点中this.retrieveUser()方法获取到了一个UserDetails实例，这个this.retrieveUser()方法也是一个抽象方法，他的实现写在了DaoAuthenticationProvider中。

![image.png](https://upload-images.jianshu.io/upload_images/8020666-ed88242ef44e01df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/8020666-1e634a806cf23221.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>接下来看this.retrieveUser()方法的实现，终于在实现中看到了调用loadUserByUsername()方法。
![image.png](https://upload-images.jianshu.io/upload_images/8020666-d23d24182b2372e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>可以看到确实进入了我们自定义的MyUserDetailsService类，这个类就不多解释了，从数据库里取数据验证而已

![image.png](https://upload-images.jianshu.io/upload_images/8020666-a92f28273f6e4fc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>回到AbstractUserDetailsAuthenticationProvider可以看到三个检查，三个检查的实现就在本类中，可以进行查看

![image.png](https://upload-images.jianshu.io/upload_images/8020666-cdc3a7d4e5aaf404.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>preAuthenticationCheck检查

![image.png](https://upload-images.jianshu.io/upload_images/8020666-c576b8db02b64d8a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>this.additionalAuthenticationChecks密码检查，实现类在DaoAuthenticationProvider中

![image.png](https://upload-images.jianshu.io/upload_images/8020666-444912c2e97c7afb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>this.postAuthenticationChecks()还是对UserDetails接口中剩下的一个布尔值进行检查

![image.png](https://upload-images.jianshu.io/upload_images/8020666-ff329ea1b6616ffb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>所有检查通过之后，认为认证成功，拿着认证成功的这些信息进入this.createSuccessAuthentication()

![image.png](https://upload-images.jianshu.io/upload_images/8020666-73a5663179f305e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/8020666-19a46c67c28a8eec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>可以看到走的确实是三个参数的构造函数，如下图，通过super.setAuthenticated(true)设置认证状态为成功

![image.png](https://upload-images.jianshu.io/upload_images/8020666-9846fcbe2c605c5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>然后DaoAuthenticationProvider返回一个认证成功的Authentication，经过认证的Authentication会沿着认证的流程返回去，一直返回到UsernamepasswordAuthenticationFilter.

![image.png](https://upload-images.jianshu.io/upload_images/8020666-99fb3f8fd00d1356.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/8020666-91babccb475d60dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/8020666-3fc3aa0a9a30e6e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>接下来就调用认证成功处理器进行处理，认证信息设置到线程中，用与session共享认证结果

![image.png](https://upload-images.jianshu.io/upload_images/8020666-7cdf705ff84ffb1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/8020666-fb3451c20d8885ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>登陆成功

![image.png](https://upload-images.jianshu.io/upload_images/8020666-02c77481448b34e0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
