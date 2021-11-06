




# 第二部份：Spring MVC 高級技術

## 第 1 節 攔截器(Inteceptor)使⽤

### 1.1 監聽器、過濾器和攔截器對⽐
- Servlet：處理Request請求和Response響應
- 過濾器（Filter）：對Request請求起到過濾的作⽤，作⽤在Servlet之前，如果配置為 /* 可以對所有的資源訪問（servlet、js/css靜態資源等）進⾏過濾處理
- 監聽器（Listener）：實現了javax.servlet.ServletContextListener 接⼝的服務器端組件，它隨Web應⽤的啟動⽽啟動，只初始化⼀次，然後會⼀直運⾏監視，隨Web應⽤的停⽌⽽銷毀
	- 作⽤⼀：做⼀些初始化⼯作，web應⽤中spring容器啟動ContextLoaderListener
	- 作⽤⼆：監聽web中的特定事件，⽐如HttpSession,ServletRequest的創建和銷毀；變量的創建、銷毀和修改等。可以在某些動作前後增加處理，實現監控，⽐如統計在線⼈數，利⽤HttpSessionLisener等。

- 攔截器（Interceptor）：是SpringMVC、Struts等表現層框架⾃⼰的，不會攔截jsp/html/css/image的訪問等，只會攔截訪問的控制器⽅法（Handler）。
	從配置的⻆度也能夠總結髮現：serlvet、filter、listener是配置在web.xml中的，⽽interceptor是配置在表現層框架⾃⼰的配置⽂件中的
	 - 在Handler業務邏輯執⾏之前攔截⼀次
	 - 在Handler邏輯執⾏完畢但未跳轉⻚⾯之前攔截⼀次
	 - 在跳轉⻚⾯之後攔截⼀次

![[Pasted image 20211023175259.png]]


### 1.2 攔截器的執⾏流程
在運⾏程序時，攔截器的執⾏是有⼀定順序的，該順序與配置⽂件中所定義的攔截器的順序相關。單個攔截器，在程序中的執⾏流程如下圖所示：
![[Pasted image 20211023175318.png]]

1) 程序先執⾏preHandle()⽅法，如果該⽅法的返回值為true，則程序會繼續向下執⾏處理器中的⽅法，否則將不再向下執⾏。
2) 在業務處理器（即控制器Controller類）處理完請求後，會執⾏postHandle()⽅法，然後會通過DispatcherServlet向客戶端返迴響應。
3) 在DispatcherServlet處理完請求後，才會執⾏afterCompletion()⽅法。

### 1.3 多個攔截器的執⾏流程
多個攔截器（假設有兩個攔截器Interceptor1和Interceptor2，並且在配置⽂件中， Interceptor1攔截器配置在前），在程序中的執⾏流程如下圖所示：
![[Pasted image 20211023175525.png]]
從圖可以看出，當有多個攔截器同時⼯作時，它們的preHandle()⽅法會按照配置⽂件中攔截器的配置順序執⾏，⽽它們的postHandle()⽅法和afterCompletion()⽅法則會按照配置順序的反序執⾏。

**示例代碼**

⾃定義SpringMVC攔截器
```java
package com.lagou.edu.interceptor;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
/**
* ⾃定义springmvc拦截器
*/
public class MyIntercepter01 implements HandlerInterceptor {
 /**
 * 会在handler⽅法业务逻辑执⾏之前执⾏
 * 往往在这⾥完成权限校验⼯作
 * @param request
 * @param response
 * @param handler
 * @return 返回值boolean代表是否放⾏，true代表放⾏，false代表中⽌
 * @throws Exception
 */
 @Override
 public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
 System.out.println("MyIntercepter01 preHandle......");
  return true;
 }
 /**
 * 会在handler⽅法业务逻辑执⾏之后尚未跳转⻚⾯时执⾏
 * @param request
 * @param response
 * @param handler
 * @param modelAndView 封装了视图和数据，此时尚未跳转⻚⾯呢，你可以在这⾥针对返回的
数据和视图信息进⾏修改
 * @throws Exception
 */
 @Override
 public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
  System.out.println("MyIntercepter01 postHandle......");
 }
 /**
 * ⻚⾯已经跳转渲染完毕之后执⾏
 * @param request
 * @param response
 * @param handler
 * @param ex 可以在这⾥捕获异常
 * @throws Exception
 */
 @Override
 public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
  System.out.println("MyIntercepter01 afterCompletion......");
 }
}
```

註冊SpringMVC攔截器
```xml
<mvc:interceptors>
 <!--拦截所有handler-->
 <!--<bean class="com.lagou.edu.interceptor.MyIntercepter01"/>-->
 
 <mvc:interceptor>
	 <!--配置当前拦截器的url拦截规则，**代表当前⽬录下及其⼦⽬录下的所有url-->
	 <mvc:mapping path="/**"/>
	 <!--exclude-mapping可以在mapping的基础上排除⼀些url拦截-->
	 <!--<mvc:exclude-mapping path="/demo/**"/>-->
	 <bean class="com.lagou.edu.interceptor.MyIntercepter01"/>
 </mvc:interceptor>
 
 <mvc:interceptor>
	 <mvc:mapping path="/**"/>
	 <bean class="com.lagou.edu.interceptor.MyIntercepter02"/>
 </mvc:interceptor>
 
</mvc:interceptors>
```


## 第 2 節 處理multipart形式的數據

⽂件上傳

原⽣servlet處理上傳的⽂件數據的，springmvc⼜是對serlvet的封裝

所需jar包
```xml
<!--⽂件上传所需jar坐标-->
<dependency>
 <groupId>commons-fileupload</groupId>
 <artifactId>commons-fileupload</artifactId>
 <version>1.3.1</version>
</dependency>
```

配置⽂件上傳解析器
```xml
<!--配置⽂件上传解析器，id是固定的multipartResolver-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
 <!--设置上传⼤⼩，单位字节-->
 <property name="maxUploadSize" value="1000000000"/>
</bean>
```

前端Form
```html
<%--
 1 method="post"
 2 enctype="multipart/form-data"
 3 type="file"
--%>
<form method="post" enctype="multipart/form-data" action="/demo/upload">
 <input type="file" name="uploadFile"/>
 <input type="submit" value="上传"/>
</form>
```

後台接收Handler
```java
@RequestMapping("upload")
public String upload(MultipartFile uploadFile, HttpServletRequest request) throws IOException {
 // ⽂件原名，如xxx.jpg
 String originalFilename = uploadFile.getOriginalFilename();
 // 获取⽂件的扩展名,如jpg
 String extendName = originalFilename.substring(originalFilename.lastIndexOf(".") + 1, originalFilename.length());
 String uuid = UUID.randomUUID().toString();
 // 新的⽂件名字
 String newName = uuid + "." + extendName;
 String realPath = request.getSession().getServletContext().getRealPath("/uploads");
 // 解决⽂件夹存放⽂件数量限制，按⽇期存放
 String datePath = new SimpleDateFormat("yyyy-MM-dd").format(new Date());
 File floder = new File(realPath + "/" + datePath);
 if(!floder.exists()) {
  floder.mkdirs();
 }
 uploadFile.transferTo(new File(floder,newName));
 return "success";
}

```


## 第 3 节 在控制器中处理异常

```java
package com.lagou.edu.controller;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.servlet.ModelAndView;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
// 可以让我们优雅的捕获所有Controller对象handler⽅法抛出的异常
@ControllerAdvice
public class GlobalExceptionResolver {
 @ExceptionHandler(ArithmeticException.class)
 public ModelAndView handleException(ArithmeticException exception,
HttpServletResponse response) {
 ModelAndView modelAndView = new ModelAndView();
 modelAndView.addObject("msg",exception.getMessage());
 modelAndView.setViewName("error");
 return modelAndView;
 }
}
```


## 第 4 節 基於Flash屬性的跨重定向請求數據傳遞

重定向時請求參數會丟失，我們往往需要重新攜帶請求參數，我們可以進⾏⼿動參數拼接如下：
```java
return "redirect:handle01?name=" + name;
```

但是上述拼接參數的⽅法屬於get請求，攜帶參數⻓度有限制，參數安全性也不⾼，此時，我們可以使⽤SpringMVC提供的flash屬性機制，向上下⽂中添加flash屬性，框架會在session中記錄該屬性值，當跳轉到⻚⾯之後框架會⾃動刪除flash屬性，不需要我們⼿動刪除，通過這種⽅式進⾏重定向參數傳遞，參數⻓度和安全性都得到了保障，如下：
```java
/**
 * SpringMVC 重定向时参数传递的问题
 * 转发：A 找 B 借钱400，B没有钱但是悄悄的找到C借了400块钱给A
 * url不会变,参数也不会丢失,⼀个请求
 * 重定向：A 找 B 借钱400，B 说我没有钱，你找别⼈借去，那么A ⼜带着400块的借钱需求找到
C
 * url会变,参数会丢失需要重新携带参数,两个请求
 */
 @RequestMapping("/handleRedirect")
 public String handleRedirect(String name,RedirectAttributes redirectAttributes) {
  //return "redirect:handle01?name=" + name; // 拼接参数安全性、参数⻓度都有局限
  // addFlashAttribute⽅法设置了⼀个flash类型属性，该属性会被暂存到session中，在跳转到⻚⾯之后该属性销毁
  redirectAttributes.addFlashAttribute("name",name);
  return "redirect:handle01";
 }

```
































