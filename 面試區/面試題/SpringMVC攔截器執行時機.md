---
界: Spring Framework
門: SpringMVC
綱: 
tags: ["#Spring","#SpringMVC","#MVC","#BQ"]
aliases:
  - 
date: 2021-09-08
---

-   程式先執行preHandle()方法，如果該方法的返回值為true，則程式繼續向下執行處理器中的方法， 否則將不再向下執行。
-   在業務處理器（控制器Controller類）處理完請求后，會執行postHandle()方法，然後通過 DispatcherServlet向客戶端返回響應。
-   DispatcherServlet處理完請求后，才會執行afterCompletion()方法。