---
title: session与cookie
date: 2018-09-27 11:15:43
tags: J2EE
---


需要对[JavaEE规范](https://docs.oracle.com/javaee/7/tutorial/index.html)有较为完整的认识

* servlet是java定义的用于开发服务端的接口，如HttpServletRequest、 Cookie、HttpSession等，但是实现由具体容器决定，比如tomcat、jetty等
<!-- more --> 
* 我们知道session在client是以JSESSIONID放入浏览器cookie中的

    ```
    Request Cookies
    JSESSIONID	1i6526oj6gvohlvm96apc68wc
    Response Cookies
    ```

* session的产生

    通常说一次连接建立，默认服务器端就产生了一个会话，但事实上我们如果不手动调用 session创建代码，浏览器端并没有显示cookie，更看不到JSESSIONID，浏览器不显示cookie，cookie不是浏览器的东西吗？
    
    ```
    @GetMapping("/versions")
    public ResponseEntity selectAll(@RequestParam("device_type") String device_type, HttpServletRequest request) {
        System.out.println(request.getClass());

        Cookie[] cookies = request.getCookies();
        if(cookies !=null) {
            Arrays.stream(cookies).forEach((cookie) ->
                    log.info(cookie.getName() + " : " + cookie.getValue())
            );
        }


        OTAVersionExample example = new OTAVersionExample();
        example.createCriteria().andDevice_typeEqualTo(device_type);
        List<OTAVersion> otaVersionList = otaVersionMapper.selectByExample(example);
        return Ok.newOk(otaVersionList, HttpStatus.OK);
    }

    ```
    
	<p></p>
    
* cookie定义(wikipedia)
    > <p>An HTTP cookie (also called web cookie, Internet cookie, browser cookie, or simply cookie) is a small piece of data sent from a website and stored on the user's computer by the user's web browser while the user is browsing.
        </p>

      cookie是由服务端发送过来，浏览器存储的数据片 
         
	<p></p>
	
    * 结论：

        session 需要服务端新建后，将id放入cookie里发送到客户端，客户端下次请求放入头部传到服务器端来验证是否是统一会话
        修改代码
        
        ```
        @GetMapping("/versions")
        public ResponseEntity selectAll(@RequestParam("device_type") String device_type, HttpServletRequest request) {
            System.out.println(request.getClass()); // class org.eclipse.jetty.server.Request

            HttpSession session = request.getSession(); // 手动创建session
            Cookie[] cookies = request.getCookies();
            if(cookies !=null) {
                Arrays.stream(cookies).forEach((cookie) ->
                        log.info(cookie.getName() + " : " + cookie.getValue())
                );
            }


            OTAVersionExample example = new OTAVersionExample();
            example.createCriteria().andDevice_typeEqualTo(device_type);
            List<OTAVersion> otaVersionList = otaVersionMapper.selectByExample(example);
            return Ok.newOk(otaVersionList, HttpStatus.OK);
        }
        ```
        
        首次请求：
        
        ```
        Request Cookies
        Response Cookies
        JSESSIONID	lelbt581q3ytnl16p06mpl0r
        ```
        
        刷新后：
        
        ```
        Request Cookies
        JSESSIONID	lelbt581q3ytnl16p06mpl0r
        Response Cookies

        ```
        
        只是手动创建了session，JSESSIONID是如何放入cookie发送到客户端的呢？跟踪代码
        `org.eclipse.jetty.server.Request`

        ```
        public class Request implements HttpServletRequest
        {

            ....

            @Override
            public HttpSession getSession()
            {
                return getSession(true);
            }

            @Override
            public HttpSession getSession(boolean create)
            {
                if (_session != null)
                {
                    if (_sessionManager != null && !_sessionManager.isValid(_session))
                        _session = null;
                    else
                        return _session;
                }

                if (!create)
                    return null;

                if (getResponse().isCommitted())
                    throw new IllegalStateException("Response is committed");

                if (_sessionManager == null)
                    throw new IllegalStateException("No SessionManager");

                _session = _sessionManager.newHttpSession(this);
                HttpCookie cookie = _sessionManager.getSessionCookie(_session,getContextPath(),isSecure()); //为session添加cookie
                if (cookie != null)
                    _channel.getResponse().addCookie(cookie); //看到了添加cookie到response消息中

                return _session;
            }
        ```
        
        ```
            @Override
            public HttpCookie getSessionCookie(HttpSession session, String contextPath, boolean requestIsSecure)
            {
                if (isUsingCookies())
                {
                    String sessionPath = (_cookieConfig.getPath()==null) ? contextPath : _cookieConfig.getPath();
                    sessionPath = (sessionPath==null||sessionPath.length()==0) ? "/" : sessionPath;
                    String id = getNodeId(session);
                    HttpCookie cookie = null;
                    if (_sessionComment == null)
                    {
                        cookie = new HttpCookie(
                                                _cookieConfig.getName(), // JSESSIONID
                                                id,
                                                _cookieConfig.getDomain(),
                                                sessionPath,
                                                _cookieConfig.getMaxAge(),
                                                _cookieConfig.isHttpOnly(),
                                                _cookieConfig.isSecure() || (isSecureRequestOnly() && requestIsSecure));
                    }
                    else
                    {
                        cookie = new HttpCookie(
                                                _cookieConfig.getName(),
                                                id,
                                                _cookieConfig.getDomain(),
                                                sessionPath,
                                                _cookieConfig.getMaxAge(),
                                                _cookieConfig.isHttpOnly(),
                                                _cookieConfig.isSecure() || (isSecureRequestOnly() && requestIsSecure),
                                                _sessionComment,
                                                1);
                    }

                    return cookie;
                }
                return null;
            }
        ```

