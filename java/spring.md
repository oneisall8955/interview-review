# spring 相关

## Spring AOP原理作用
（待完善）
Spring AOP是基于动态代理的。
AOP(Aspect-Oriented Programming:面向切面编程)能够将那些与业务无关，
却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，
便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

## Spring Boot请求过程
还是原本Spring MVC的请求过程。
1. 客户端（浏览器）发送请求，直接请求到 DispatcherServlet。
2. DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。
3. 解析到对应的 Handler（Controller）后，开始由 HandlerAdapter 适配器处理。
4. HandlerAdapter 会根据 Handler 来调用真正的处理器开处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。
6. ViewResolver 会根据逻辑 View 查找实际的 View。
7. DispatcherServlet 把返回的 Model 传给 View（视图渲染）。
8. 把 View 返回给请求者（浏览器）

## Spring中用到了哪些设计模式
