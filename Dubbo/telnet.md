```shell
telnet localhost port

//查看服务接口
ls

//查看服务接口方法
ls 接口名

//调用服务
invoke 接口名.方法名(参数)
```

---

Akulaku

```
telnet localhost 20880
invoke com.akulaku.service.goods.api.base.IllegalWordService.updateIllegalWordTask({taskId:1, status:2, scanCount:5000, intervalTime:54321})
```

