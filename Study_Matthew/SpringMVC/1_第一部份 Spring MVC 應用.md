---
界: Spring Framework
门: Spring
纲: 
tags: ["#SpringFramework/#SpringMVC","#MVC","#Note"]
aliases:
  - 
date: {{DATE:YYYY-MM-DD}}
---
# 第一部份 Spring MVC 應用

## 第 1 節 Spring MVC 簡介

### 1.1 MVC 體系結構
**三層架構**

我們的開發架構⼀般都是基於兩種形式，⼀種是 C/S 架構，也就是客戶端/服務器；另⼀種是 B/S 架構，也就是瀏覽器服務器。在 JavaEE 開發中，⼏乎全都是基於 B/S 架構的開發。那麼在 B/S 架構中，系
統標準的三層架構包括：表現層、業務層、持久層。三層架構在我們的實際開發中使⽤的⾮常多，所以我們課程中的案例也都是基於三層架構設計的。

三層架構中，每⼀層各司其職，接下來我們就說說每層都負責哪些⽅⾯：

- 表現層 ：
	
	也就是我們常說的 Web 層。它負責接收客戶端請求，向客戶端響應結果，通常客戶端使⽤ http 協議請求 Web 層，web 需要接收 http 請求，完成 http 響應。
	
	表現層包括展示層和控制層：控制層負責接收請求，展示層負責結果的展示。
	
	表現層依賴業務層，接收到客戶端請求⼀般會調⽤業務層進⾏業務處理，並將處理結果響應給客戶端。
	
	表現層的設計⼀般都使⽤ MVC 模型。 （MVC 是表現層的設計模型，和其他層沒有關係）

- 業務層 ：
	
	也就是我們常說的 service 層。它負責業務邏輯處理，和我們開發項⽬的需求息息相關。 Web 層依賴業務層，但是業務層不依賴 Web 層。
	
	業務層在業務處理時可能會依賴持久層，如果要對數據持久化需要保證事務⼀致性。 （也就是我們說的， 事務應該放到業務層來控制）

- 持久層 ：
	
	也就是我們是常說的 dao 層。負責數據持久化，包括數據層即數據庫和數據訪問層，數據庫是對數據進⾏持久化的載體，數據訪問層是業務層和持久層交互的接⼝，業務層需要通過數據訪問層將數據持久化到數據庫中。通俗的講，持久層就是和數據庫交互，對數據庫表進⾏增刪改查的。

**MVC 設計模式**

MVC 全名是 Model View Controller，是 模型(model)－視圖(view)－控制器(controller) 的縮寫， 是⼀種⽤於設計創建 Web 應⽤程序表現層的模式。 MVC 中每個部分各司其職：

- Model（模型）：模型包含業務模型和數據模型，數據模型⽤於封裝數據，業務模型⽤於處理業務。
- View（視圖）： 通常指的就是我們的 jsp 或者 html。作⽤⼀般就是展示數據的。通常視圖是依據模型數據創建的。
- Controller（控制器）： 是應⽤程序中處理⽤戶交互的部分。作⽤⼀般就是處理程序邏輯的。

MVC 提倡：每⼀層只編寫⾃⼰的東⻄，不編寫任何其他的代碼；分層是為了解耦，解耦是為了維護⽅便和分⼯協作。

### 1.2 Spring MVC 是什麼？
SpringMVC 全名叫 Spring Web MVC，是⼀種基於 Java 的實現 MVC 設計模型的請求驅動類型的輕量級 Web 框架，屬於 SpringFrameWork 的後續產品。
![[Pasted image 20211016190440.png]]

SpringMVC 已經成為 ⽬前最主流的 MVC 框架 之⼀，並且 隨著 Spring3.0 的發布，全⾯超越 Struts2，成為最優秀的 MVC 框架。

servlet、struts 實現接⼝、springmvc 中要讓⼀個 java 類能夠處理請求只需要添加註解就 ok

它通過⼀套註解，讓⼀個簡單的 Java 類成為處理請求的控制器，⽽⽆須實現任何接⼝。同時它還⽀持 RESTful 編程⻛格的請求。

總之：Spring MVC 和 Struts2 ⼀樣，都是 為了解決表現層問題 的 Web 框架，它們都是基於 MVC 設計模式的。 ⽽這些表現層框架的主要職責就是處理前端 HTTP 請求。

Spring MVC 本質可以認為是對 servlet 的封裝，簡化了我們 serlvet 的開發

作⽤：1）接收請求 2）返迴響應，跳轉⻚⾯
![[Pasted image 20211016190550.png]]

## 第 2 節 Spring Web MVC ⼯作流程

需求：前端瀏覽器請求 url：http://localhost:8080/demo/handle01，前端⻚⾯顯示後台服務器的時間

開發過程

1）配置 DispatcherServlet 前端控制器

2）開發處理具體業務邏輯的 Handler（@Controller、@RequestMapping）

3）xml 配置⽂件配置 controller 掃描，配置 springmvc 三⼤件

4）將 xml ⽂件路徑告訴 springmvc（DispatcherServlet）

### 2.1 Spring MVC 請求處理流程
![[Pasted image 20211016191311.png]]

**流程說明**

第⼀步：⽤戶發送請求⾄前端控制器 DispatcherServlet

第⼆步：DispatcherServlet 收到請求調⽤ HandlerMapping 處理器映射器

第三步：處理器映射器根據請求 Url 找到具體的 Handler（後端控制器），⽣成處理器對象及處理器攔截器(如果 有則⽣成)⼀並返回 DispatcherServlet

第四步：DispatcherServlet 調⽤ HandlerAdapter 處理器適配器去調⽤ Handler

第五步：處理器適配器執⾏ Handler

第六步：Handler 執⾏完成給處理器適配器返回 ModelAndView

第七步：處理器適配器向前端控制器返回 ModelAndView，ModelAndView 是 SpringMVC 框架的⼀個底層對 象，包括 Model 和 View

第⼋步：前端控制器請求視圖解析器去進⾏視圖解析，根據邏輯視圖名來解析真正的視圖。

第九步：視圖解析器向前端控制器返回 View

第⼗步：前端控制器進⾏視圖渲染，就是將模型數據（在 ModelAndView 對像中）填充到 request 域

第⼗⼀步：前端控制器向⽤戶響應結果

### 2.2 Spring MVC 九⼤組件

- HandlerMapping（處理器映射器）

	HandlerMapping 是⽤來查找 Handler 的，也就是處理器，具體的表現形式可以是類，也可以是⽅法。 ⽐如，標註了@RequestMapping 的每個⽅法都可以看成是⼀個 Handler。 Handler 負責具體實際的請求處理，在請求到達後，HandlerMapping 的作⽤便是找到請求相應的處理器 Handler 和 Interceptor.

- HandlerAdapter（處理器適配器）

	HandlerAdapter 是⼀個適配器。因為 Spring MVC 中 Handler 可以是任意形式的，只要能處理請求即可。但是把請求交給 Servlet 的時候，由於 Servlet 的⽅法結構都是 doService(HttpServletRequest req,HttpServletResponse resp)形式的，要讓固定的 Servlet 處理⽅法調⽤ Handler 來進⾏處理，便是 HandlerAdapter 的職責。

- HandlerExceptionResolver

	HandlerExceptionResolver ⽤於處理 Handler 產⽣的異常情況。它的作⽤是根據異常設置 ModelAndView，之後交給渲染⽅法進⾏渲染，渲染⽅法會將 ModelAndView 渲染成⻚⾯。

- ViewResolver

	ViewResolver 即視圖解析器，⽤於將 String 類型的視圖名和 Locale 解析為 View 類型的視圖，只有⼀個 resolveViewName()⽅法。從⽅法的定義可以看出，Controller 層返回的 String 類型視圖名 viewName 最終會在這⾥被解析成為 View。 View 是⽤來渲染⻚⾯的，也就是說，它會將程序返回的參數和數據填⼊模板中，⽣成 HTML ⽂件。 ViewResolver 在這個過程主要完成兩件事情：ViewResolver 找到渲染所⽤的模板（第⼀件⼤事）和所⽤的技術（第⼆件⼤事，其實也就是找到視圖的類型，如 JSP）並填⼊參數。默認情況下，Spring MVC 會⾃動為我們配置⼀個 InternalResourceViewResolver,是針對 JSP 類型視圖的。

- RequestToViewNameTranslator

	RequestToViewNameTranslator 組件的作⽤是從請求中獲取 ViewName.因為 ViewResolver 根據 ViewName 查找 View，但有的 Handler 處理完成之後,沒有設置 View，也沒有設置 ViewName，便要通過這個組件從請求中查找 ViewName。

- LocaleResolver

	ViewResolver 組件的 resolveViewName ⽅法需要兩個參數，⼀個是視圖名，⼀個是 Locale。LocaleResolver ⽤於從請求中解析出 Locale，⽐如中國 Locale 是 zh-CN，⽤來表示⼀個區域。這個組件也是 I18N 的基礎。

- ThemeResolver

	ThemeResolver 組件是⽤來解析主題的。主題是樣式、圖⽚及它們所形成的顯示效果的集合。Spring MVC 中⼀套主題對應⼀個 properties ⽂件，⾥⾯存放著與當前主題相關的所有資源，如圖⽚、CSS 樣式等。創建主題⾮常簡單，只需準備好資源，然後新建⼀個“主題名.properties”並將資源設置進去，放在 classpath 下，之後便可以在⻚⾯中使⽤了。 SpringMVC 中與主題相關的類有 ThemeResolver、ThemeSource 和 Theme。 ThemeResolver 負責從請求中解析出主題名，ThemeSource 根據主題名找到具體的主題，其抽像也就是 Theme，可以通過 Theme 來獲取主題和具體的資源。

- MultipartResolver

	MultipartResolver ⽤於上傳請求，通過將普通的請求包裝成 MultipartHttpServletRequest 來實現。 MultipartHttpServletRequest 可以通過 getFile() ⽅法 直接獲得⽂件。如果上傳多個⽂件，還可以調⽤ getFileMap()⽅法得到 Map<FileName，File> 這樣的結構，MultipartResolver 的作⽤就是封裝普通的請求，使其擁有⽂件上傳的功能。

- FlashMapManager

	FlashMap ⽤於重定向時的參數傳遞，⽐如在處理⽤戶訂單時候，為了避免重複提交，可以處理完 post 請求之後重定向到⼀個 get 請求，這個 get 請求可以⽤來顯示訂單詳情之類的信息。這樣做雖然可以規避⽤戶重新提交訂單的問題，但是在這個⻚⾯上要顯示訂單的信息，這些數據從哪⾥來獲得呢？因為重定向時麼有傳遞參數這⼀功能的，如果不想把參數寫進 URL（不推薦），那麼就可以通過 FlashMap 來傳遞。只需要在重定向之前將要傳遞的數據寫⼊請求（可以通過 ServletRequestAttributes.getRequest()⽅法獲得）的屬性 OUTPUT_FLASH_MAP_ATTRIBUTE 中，這樣在重定向之後的 Handler 中 Spring 就會⾃動將其設置到 Model 中，在顯示訂單信息的⻚⾯上就可以直接從 Model 中獲取數據。 FlashMapManager 就是⽤來管理 FalshMap 的。

**SpringMVC 的 url-pattern 配置及原理解析**
- 參考教學影片第 9 堂
- 專案名稱：springmvc-demo

**數據輸出機制之 Model、Map 及 ModelMap**
- 參考教學影片第 10 堂
- 專案名稱：springmvc-demo


## 第 3 節 請求參數綁定（串講）
請求參數綁定：說⽩了 SpringMVC 如何接收請求參數

http 協議（超⽂本傳輸協議）

原⽣ servlet 接收⼀個整型參數：

1) String ageStr = request.getParameter("age");
2) Integer age = Integer.parseInt(ageStr);

SpringMVC 框架對 Servlet 的封裝，簡化了 servlet 的很多操作

SpringMVC 在接收整型參數的時候，直接在 Handler ⽅法中聲明形參即可

@RequestMapping("xxx")

public String handle(Integer age) {

System.out.println(age);

}

參數綁定：取出參數值綁定到 handler ⽅法的形參上

- 默認⽀持 Servlet API 作為⽅法參數

	當需要使⽤ HttpServletRequest、HttpServletResponse、HttpSession 等原⽣ servlet 對象時，直接在 handler ⽅法中形參聲明使⽤即可。
	```java
	/**
	 *
	 * SpringMVC 对原⽣ servlet API 的⽀持 url：/demo/handle02?id=1
	 *
	 * 如果要在 SpringMVC 中使⽤ servlet 原⽣对象，⽐如
	HttpServletRequest\HttpServletResponse\HttpSession，直接在 Handler ⽅法形参中声
	明使⽤即可
	 *
	 */
	 @RequestMapping("/handle02")
	 public ModelAndView handle02(HttpServletRequest request, HttpServletResponse response,HttpSession session) {
		 String id = request.getParameter("id");
		 Date date = new Date();
		 ModelAndView modelAndView = new ModelAndView();
		 modelAndView.addObject("date",date);
		 modelAndView.setViewName("success");
		 return modelAndView;
	 }
	```

- 綁定簡單類型參數

	簡單數據類型：⼋種基本數據類型及其包裝類型

	參數類型推薦使⽤包裝數據類型，因為基礎數據類型不可以為 null

	整型：Integer、int

	字符串：String

	單精度：Float、float

	雙精度：Double、double

	布爾型：Boolean、boolean

	說明：對於布爾類型的參數，請求的參數值為 true 或 false。或者 1 或 0

	注意：綁定簡單數據類型參數，只需要直接聲明形參即可（形參參數名和傳遞的參數名要保持⼀致，建議 使⽤包裝類型，當形參參數名和傳遞參數名不⼀致時可以使⽤@RequestParam 註解進⾏⼿動映射）

	```java
	/*
	 * SpringMVC 接收简单数据类型参数 url：/demo/handle03?id=1
	 *
	 * 注意：接收简单数据类型参数，直接在 handler ⽅法的形参中声明即可，框架会取出参数值
	然后绑定到对应参数上
	 * 要求：传递的参数名和声明的形参名称保持⼀致
	 */
	 @RequestMapping("/handle03")
	 public ModelAndView handle03(@RequestParam("ids") Integer id,Boolean flag) {
		 Date date = new Date();
		 ModelAndView modelAndView = new ModelAndView();
		 modelAndView.addObject("date",date);
		 modelAndView.setViewName("success");
		 return modelAndView;
	 }
	```

- 綁定 Pojo 類型參數
	```java
	/*
	 * SpringMVC 接收 pojo 类型参数 url：/demo/handle04?id=1&username=zhangsan
	 *
	 * 接收 pojo 类型参数，直接形参声明即可，类型就是 Pojo 的类型，形参名⽆所谓
	 * 但是要求传递的参数名必须和 Pojo 的属性名保持⼀致
	 */
	 @RequestMapping("/handle04")
	 public ModelAndView handle04(User user) {
	 Date date = new Date();
	```

- 綁定 Pojo 包裝對象參數

	包裝類型 QueryVo
	
	```java
	package com.lagou.edu.pojo;
	/**
	* @author 应癫
	*/
	public class QueryVo {
	 private String mail;
	 private String phone;
	 // 嵌套了另外的 Pojo 对象
	 private User user;
	 public String getMail() {
	 return mail;
	 }
	 public void setMail(String mail) {
	 this.mail = mail;
	 }
	 public String getPhone() {
	 return phone;
	 }
	 public void setPhone(String phone) {
	 this.phone = phone;
	 }
	 public User getUser() {
	 return user;
	 }
	 public void setUser(User user) {
	 this.user = user;
	 }
	}
	```

	Handler ⽅法
	```java
	/*
	 * SpringMVC 接收 pojo 包装类型参数 url：/demo/handle05?
	user.id=1&user.username=zhangsan
	 * 不管包装 Pojo 与否，它⾸先是⼀个 pojo，那么就可以按照上述 pojo 的要求来
	 * 1、绑定时候直接形参声明即可
	 * 2、传参参数名和 pojo 属性保持⼀致，如果不能够定位数据项，那么通过属性名 + "." 的
	⽅式进⼀步锁定数据
	 *
	 */
	 @RequestMapping("/handle05")
	 public ModelAndView handle05(QueryVo queryVo) {
	 Date date = new Date();
	 ModelAndView modelAndView = new ModelAndView();
	 modelAndView.addObject("date",date);
	 modelAndView.setViewName("success");
	 return modelAndView;
	 }
	```


## 第 4 節 對 Restful ⻛格請求⽀持
- REST ⻛格請求是什麼樣的？
- springmvc 對 REST ⻛格請求到底提供了怎樣的⽀持
	
	是⼀個註解的使⽤@PathVariable，可以幫助我們從 uri 中取出參數


### 4.1 什麼是 RESTful
Restful 是⼀種 Web 軟件架構⻛格，它不是標準也不是協議，它倡導的是⼀個資源定位及資源操作的⻛格。

**什麼是 REST：**

REST（英⽂：Representational State Transfer，簡稱 REST）描述了⼀個架構樣式的⽹絡系統， ⽐如 Web 應⽤程序。它⾸次出現在 2000 年 Roy Fielding 的博⼠論⽂中，他是 HTTP 規範的主要編寫者之⼀。在⽬前主流的三種 Web 服務交互⽅案中，REST 相⽐於 SOAP（Simple Object Access protocol，簡單對象訪問協議）以及 XML-RPC 更加簡單明了，⽆論是對 URL 的處理還是對 Payload 的編碼，REST 都傾向於⽤更加簡單輕量的⽅法設計和實現。值得注意的是 REST 並沒有⼀個明確的標準，⽽更像是⼀種設計的⻛格。

它本身並沒有什麼實⽤性，其核⼼價值在於如何設計出符合 REST ⻛格的⽹絡接⼝。

資源 表現層 狀態轉移

**Restful 的優點**

它結構清晰、符合標準、易於理解、擴展⽅便，所以正得到越來越多⽹站的採⽤。

**Restful 的特性**

資源（Resources）：⽹絡上的⼀個實體，或者說是⽹絡上的⼀個具體信息。

它可以是⼀段⽂本、⼀張圖⽚、⼀⾸歌曲、⼀種服務，總之就是⼀個具體的存在。可以⽤⼀個 URI（統⼀資源定位符）指向它，每種資源對應⼀個特定的 URI 。要獲取這個資源，訪問它的 URI 就可以，因此 URI 即為每⼀個資源的獨⼀⽆⼆的識別符。

表現層（Representation）：把資源具體呈現出來的形式，叫做它的表現層 （Representation）。 ⽐如，⽂本可以⽤ txt 格式表現，也可以⽤ HTML 格式、XML 格式、JSON 格式表現，甚⾄可以採⽤⼆進制格式。

狀態轉化（State Transfer）：每發出⼀個請求，就代表了客戶端和服務器的⼀次交互過程。

HTTP 協議，是⼀個⽆狀態協議，即所有的狀態都保存在服務器端。因此，如果客戶端想要操作服務器， 必須通過某種⼿段，讓服務器端發⽣“狀態轉化”（State Transfer）。 ⽽這種轉化是建⽴在表現層之上的，所以就是 “ 表現層狀態轉化” 。具體說， 就是 HTTP 協議⾥⾯，四個表示操作⽅式的動詞：GET 、POST 、PUT 、DELETE 。它們分別對應四種基本操作：GET ⽤來獲取資源，POST ⽤來新建資源，PUT ⽤來更新資源，DELETE ⽤來刪除資源。

**RESTful 的示例：**

rest 是⼀個 url 請求的⻛格，基於這種⻛格設計請求的 url

沒有 REST 的話，原有的 url 設計

http://localhost:8080/user/queryUserById.action?id=3

url 中定義了動作（操作），參數具體鎖定到操作的是誰

有了 REST ⻛格之後

rest 中，認為互聯⽹中的所有東⻄都是資源，既然是資源就會有⼀個唯⼀的 uri 標識它，代表它

http://localhost:8080/user/3 代表的是 id 為 3 的那個⽤戶記錄（資源）

鎖定資源之後如何操作它呢？常規操作就是增刪改查

根據請求⽅式不同，代表要做不同的操作

get 查詢，獲取資源

post 增加，新建資源

put 更新

delete 刪除資源

rest ⻛格帶來的直觀體現：就是傳遞參數⽅式的變化，參數可以在 uri 中了

/account/1 HTTP GET ：得到 id = 1 的 account

/account/1 HTTP DELETE：刪除 id = 1 的 account

/account/1 HTTP PUT：更新 id = 1 的 account

URL：資源定位符，通過 URL 地址去定位互聯⽹中的資源（抽象的概念，⽐如圖⽚、視頻、app 服務等）。

**RESTful ⻛格 URL**：互聯⽹所有的事物都是資源，要求 URL 中只有表示資源的名稱，沒有動詞。

**RESTful ⻛格資源操作**：使⽤ HTTP 請求中的 method ⽅法 put、delete、post、get 來操作資源。分別對應添加、刪除、修改、查詢。不過⼀般使⽤時還是 post 和 get。 put 和 delete ⼏乎不使⽤。

**RESTful ⻛格資源表述**：可以根據需求對 URL 定位的資源返回不同的表述（也就是返回數據類型，⽐如 XML、JSON 等數據格式）。

Spring MVC ⽀持 RESTful ⻛格請求，具體講的就是使⽤ @PathVariable 註解獲取 RESTful ⻛格的請求 URL 中的路徑變量。

- 示例代碼
	- 前端 jsp ⻚⾯
		```java
		<div>
		 <h2>SpringMVC 对 Restful ⻛格 url 的⽀持 </h2>
		 <fieldset>
		 <p> 测试⽤例：SpringMVC 对 Restful ⻛格 url 的⽀持 </p>
		 <a href="/demo/handle/15">rest_get 测试 </a>
		 <form method="post" action="/demo/handle">
		 <input type="text" name="username"/>
		 <input type="submit" value="提交 rest_post 请求"/>
		 </form>
		 <form method="post" action="/demo/handle/15/lisi">
		 <input type="hidden" name="_method" value="put"/>
		 <input type="submit" value="提交 rest_put 请求"/>
		 </form>
		 <form method="post" action="/demo/handle/15">
		 <input type="hidden" name="_method" value="delete"/>
		 <input type="submit" value="提交 rest_delete 请求"/>
		 </form>
		 </fieldset>
		</div>
		```
	- 後台 Handler ⽅法
		```java
		/*
		 * restful get /demo/handle/15
		 */
		 @RequestMapping(value = "/handle/{id}",method = {RequestMethod.GET})
		 public ModelAndView handleGet(@PathVariable("id") Integer id) {
		 Date date = new Date();
		 ModelAndView modelAndView = new ModelAndView();
		 modelAndView.addObject("date",date);
		 modelAndView.setViewName("success");
		 return modelAndView;
		 }
		 /*
		 * restful post /demo/handle
		 */
		 @RequestMapping(value = "/handle",method = {RequestMethod.POST})
		 public ModelAndView handlePost(String username) {
		 Date date = new Date();
		 ModelAndView modelAndView = new ModelAndView();
		 modelAndView.addObject("date",date);
		 modelAndView.setViewName("success");
		 return modelAndView;
		 }
		 /*
		 * restful put /demo/handle/15/lisi
		 */
		 @RequestMapping(value = "/handle/{id}/{name}",method = {RequestMethod.PUT})
		 public ModelAndView handlePut(@PathVariable("id") Integer id,@PathVariable("name") String username) {
		 Date date = new Date();
		 ModelAndView modelAndView = new ModelAndView();
		 modelAndView.addObject("date",date);
		 modelAndView.setViewName("success");
		 return modelAndView;
		 }
		 /*
		 * restful delete /demo/handle/15
		 */
		 @RequestMapping(value = "/handle/{id}",method = {RequestMethod.DELETE})
		 public ModelAndView handleDelete(@PathVariable("id") Integer id) {
		 Date date = new Date();
		 ModelAndView modelAndView = new ModelAndView();
		 modelAndView.addObject("date",date);
		 modelAndView.setViewName("success");
		 return modelAndView;
		 }
		```

	- web.xml 中配置請求⽅式過濾器（將特定的 post 請求轉換為 put 和 delete 請求）
		```xml
		<!--配置 springmvc 请求⽅式转换过滤器，会检查请求参数中是否有_method 参数，如果有就
		按照指定的请求⽅式进⾏转换-->
		 <filter>
		 <filter-name>hiddenHttpMethodFilter</filter-name>
		 <filterclass>org.springframework.web.filter.HiddenHttpMethodFilter</filterclass>
		 </filter>
		 <filter-mapping>
		 <filter-name>encoding</filter-name>
		 <url-pattern>/*</url-pattern>
		 </filter-mapping>
		 <filter-mapping>
		 <filter-name>hiddenHttpMethodFilter</filter-name>
		 <url-pattern>/*</url-pattern>
		 </filter-mapping>
		```


## 第 5 節 Ajax Json 交互

交互：兩個⽅向

1) 前端到後台：前端 Ajax 發送 JSON 格式字符串，後台直接接收為 pojo 參數，使⽤註解@RequstBody
2) 後台到前端：後台直接返回 pojo 對象，前端直接接收為 JSON 對像或者字符串，使⽤註解@ResponseBody

### 5.1 什麼是 Json
Json 是⼀種與語⾔⽆關的數據交互格式，就是⼀種字符串，只是⽤特殊符號{}內表示對象、[]內表示數組、""內是屬性或值、：表示後者是前者的值

{"name": "Michael"}可以理解為是⼀個包含 name 為 Michael 的對象

[{"name": "Michael"},{"name": "Jerry"}]就表示包含兩個對象的數組

### 5.2 @ResponseBody 註解
@responseBody 註解的作⽤是將 controller 的⽅法返回的對象通過適當的轉換器轉換為指定的格式之後，寫⼊到 response 對象的 body 區，通常⽤來返回 JSON 數據或者是 XML 數據。注意：在使⽤此註解之後不會再⾛視圖處理器，⽽是直接將數據寫⼊到輸⼊流中，他的效果等同於通過 response 對象輸出指定格式的數據。

### 5.3 分析 Spring MVC 使⽤ Json 交互
所需 jar 包
```xml
<!--json 数据交互所需 jar，start-->
<dependency>
 <groupId>com.fasterxml.jackson.core</groupId>
 <artifactId>jackson-core</artifactId>
 <version>2.9.0</version>
</dependency>
<dependency>
 <groupId>com.fasterxml.jackson.core</groupId>
 <artifactId>jackson-databind</artifactId>
 <version>2.9.0</version>
</dependency>
<dependency>
 <groupId>com.fasterxml.jackson.core</groupId>
 <artifactId>jackson-annotations</artifactId>
 <version>2.9.0</version>
</dependency>
<!--json 数据交互所需 jar，end-->
```

- 示例代码 
	- 前端 jsp ⻚⾯及 JS 代码
		```xml
		<div>
		 <h2>Ajax JSON 交互 </h2>
		 <fieldset>
		 <input type="button" id="ajaxBtn" value="ajax 提交"/>
		 </fieldset>
		</div>
		```

		```js
		$(function () {
		 $("#ajaxBtn").bind("click",function () {
			 // 发送 Ajax 请求
			 $.ajax({
				 url: '/demo/handle07',
				 type: 'POST',
				 data: '{"id":"1","name":"李四"}',
				 contentType: 'application/json;charset=utf-8',
				 dataType: 'json',
				 success: function (data) {
					alert(data.name);
				 }
			 })
		 })
		})
		```

	- 後台 Handler ⽅法
		```java
		@RequestMapping("/handle07")
		// 添加@ResponseBody 之后，不再⾛视图解析器那个流程，⽽是等同于 response 直接输出数据
		public @ResponseBody User handle07(@RequestBody User user) {
		 // 业务逻辑处理，修改 name 为张三丰
		 user.setName("张三丰");
		 return user;
		}
		```
















