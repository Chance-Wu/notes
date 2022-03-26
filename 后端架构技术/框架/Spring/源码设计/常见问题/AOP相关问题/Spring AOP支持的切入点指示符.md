>Spring AOP支持以下在切入点表达式中使用的`AspectJ切入点指示符`（PCD）：
>
>- ==execution==：匹配方法执行的连接点；==最小粒度是方法==
>
>  ```
>  execution(* com.xyz.service..*.*(..))
>  ```
>
>- ==within==：匹配指定类型内的方法执行；==最小粒度是类==
>
>  ```
>  within(com.xyz.service..*)
>  ```
>
>- `this`：匹配当前AOP代理对象类型的执行方法；注意是AOP代理对象的类型匹配，这样就可能包括引入接口也类型匹配；
>
>  ```
>  this(com.xyz.service.AccountService)
>  ```
>
>- `target`：匹配当前目标对象类型的执行方法，这样就不包括引入接口也类型匹配；
>
>  ```
>  target(com.xyz.service.AccountService)
>  ```
>
>- `args`：匹配当前执行的方法传入的参数为指定类型的执行方法；
>
>  ```
>   args(java.lang.String)
>  ```
>
>- `@target`：匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解；
>
>  ```
>  @target(org.springframework.transaction.annotation.Transactional)
>  ```
>
>- `@args`：匹配当前执行的方法传入的参数持有指定注解的执行；
>
>  ```
>  @args(com.xyz.security.Classified)
>  ```
>
>- `@within`：匹配所有持有指定注解类型内的方法；
>
>  ```
>  @within(org.springframework.transaction.annotation.Transactional)
>  ```
>
>- ==@annotation==：匹配当前执行方法持有指定注解的方法；
>
>  ```
>  @annotation(org.springframework.transaction.annotation.Transactional)
>  ```

