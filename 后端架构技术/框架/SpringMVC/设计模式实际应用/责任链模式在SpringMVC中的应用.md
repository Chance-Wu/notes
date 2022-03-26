> 责任链模式（Chain of Reponsibility）：使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这个对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

>DispatcherServlet核心方法doDispatch体现了责任链模式 request是请求，所有入参包含request的方法，都是责任链的体现。
>
>```java
>protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
>    HttpServletRequest processedRequest = request;
>    HandlerExecutionChain mappedHandler = null;
>    boolean multipartRequestParsed = false;
>
>    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
>
>    try {
>        ModelAndView mv = null;
>        Exception dispatchException = null;
>
>        try {
>            processedRequest = checkMultipart(request);
>            multipartRequestParsed = (processedRequest != request);
>            // 获取该请求的handler，每个handler实为HandlerExecutionChain，它为一个处理链，负责处理整个请求
>            mappedHandler = getHandler(processedRequest);
>            if (mappedHandler == null || mappedHandler.getHandler() == null) {
>                noHandlerFound(processedRequest, response);
>                return;
>            }
>
>            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
>
>            String method = request.getMethod();
>            boolean isGet = "GET".equals(method);
>            if (isGet || "HEAD".equals(method)) {
>                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
>                if (logger.isDebugEnabled()) {
>                    logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
>                }
>                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
>                    return;
>                }
>            }
>            // 责任链执行预处理方法，实则是将请求交给注册的请求拦截器执行
>            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
>                return;
>            }
>            // 实际的执行逻辑的部分，也就是你加了@RequestMapping注解的方法
>            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
>
>            if (asyncManager.isConcurrentHandlingStarted()) {
>                return;
>            }
>            applyDefaultViewName(processedRequest, mv);
>            // 责任链执行后处理方法，实则是将请求交给注册的请求拦截器执行
>            mappedHandler.applyPostHandle(processedRequest, response, mv);
>        }
>        catch (Exception ex) {
>            dispatchException = ex;
>        }
>        catch (Throwable err) {
>            dispatchException = new NestedServletException("Handler dispatch failed", err);
>        }
>        // 处理返回的结果，触发责任链上注册的拦截器的AfterCompletion方法，其中也用到了HandlerExecutionChain注册的handler来处理错误结果
>        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
>    }
>    catch (Exception ex) {
>        // 触发责任链上注册的拦截器的AfterCompletion方法
>        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
>    }
>    catch (Throwable err) {
>        triggerAfterCompletion(processedRequest, response, mappedHandler,
>                               new NestedServletException("Handler processing failed", err));
>    }
>    finally {
>        if (asyncManager.isConcurrentHandlingStarted()) {
>            if (mappedHandler != null) {
>                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
>            }
>        }
>        else {
>            if (multipartRequestParsed) {
>                cleanupMultipart(processedRequest);
>            }
>        }
>    }
>}
>```

