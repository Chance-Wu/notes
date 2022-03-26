##### 问题：SpringBoot事务管理异常不会滚

> 1. 启动入口需要添加事务管理开启：`@EnableTransactionManagement`
> 2. 所需要的方法上面添加事务注解：`@Transactional(rollbackFor = Exception.class)`，异常时回滚。事务注解相当于开启事务`transactionManager.commit(status);`
> 3. 添加的事务的方法，如果不进行try catch处理，则会事务回滚，如果添加了try catch事务处理，则不会回滚，此时可以手动添加事务回滚：`TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();`

> Tip：==同类中，一个没有事务的方法调用了一个有事务的方法，则不会生效，因为有一个范围。需要在调用此方法的调用函数中也要加上@Transactional注释才可以。==







































