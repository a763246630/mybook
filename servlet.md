# Servlet的生命周期

​             

Servlet_生命周期：首先加载servlet的class，实例化servlet，然后初始化servlet调用init()的方法，接着调用服务的service的方法处理doGet和doPost方法，最后是我的还有容器关闭时候调用destroy 销毁方法。

1.被创建：执行init方法，只执行一次

　　1.1Servlet什么时候被创建？

　　--默认情况下，第一次被访问时，Servlet被创建，然后执行init方法；

　　--可以配置执行Servlet的创建时机；

2.提供服务：执行service方法，执行多次

3.被销毁：当Servlet服务器正常关闭时，执行destroy方法，只执行一次

```
 1 //servlet生命周期，的三个方法，1.被创建，执行且只执行一次init方法，2.提供服务，执行service方法，执行多次 3.被销毁，当Servlet服务器正常关闭时，执行destroy方法，只执行一次。
 2     
 3     @Override
 4     public void init() throws ServletException {
 5         // TODO Auto-generated method stub
 6         super.init();
 7     }
 8     
 9     @Override
10     protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
11         // TODO Auto-generated method stub
12         super.service(req, resp);
13     }
14     
15     @Override
16     public void destroy() {
17         // TODO Auto-generated method stub
18         super.destroy();
19     }
```