websocket基于http协议的升级过程

    1.http请求头中包含Connetion:Upgrade,这个请求头表示浏览器希望升级协议
      这时候就会有下一个问题,我希望升级成什么协议?2
    2.http请求头中包含Upgrade:websocket,这个请求头表示浏览器希望升级成websocket协议
    3.服务器收到请求,如果接受升级协议请求,返回101状态码,同时响应头包含Upgrade:websocket
    为什么响应头中还需要包含Upgrade:websocket?因为请求头Upgrade中可以传多个协议,服务器需要告诉浏览器它的选择

---
