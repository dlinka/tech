WebSocket基于http协议的升级过程

    1.http请求头中包含Connetion:Upgrade //这个请求头表示浏览器希望升级协议
    2.http请求头中包含Upgrade:websocket //这个请求头表示浏览器希望升级成WebSocket协议
    3.服务器收到请求,如果接受升级协议请求,返回101状态码,同时响应头包含Upgrade:websocket
    	为什么响应头中还需要包含Upgrade:websocket?
    	因为请求头Upgrade中可以传多个协议,服务器告诉浏览器它的选择是websocket

---
